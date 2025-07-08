# Prisma Setup Runbook

## Installation

1. **Install Prisma CLI and Client:**
   ```sh
   npm install prisma --save-dev
   npm install @prisma/client
   ```

2. **Initialize Prisma:**
   ```sh
   npx prisma init
   ```
   This creates a `prisma/` directory with `schema.prisma` and a `.env` file.

## Environment Variables

- **.env**
  - Used for development database connection.
  - Example:
    ```env
    DATABASE_URL=postgresql://postgres:postgres@localhost:5432/yourdb
    ```
- **.env.test**
  - Used for test database connection.
  - Example:
    ```env
    DATABASE_URL=postgresql://postgres:postgres@localhost:5432/yourdb_test
    ```

## Parallel, Flake-Free Testing with Per-Process Schemas

To enable parallel and reliable testing with Vitest and Prisma, use a unique database schema per test process/worker. This avoids test flakiness due to shared state.

### 1. **Create the Test Fixture**

- Create a file at `test/prisma-fixture.ts` with logic to:
  - Create a unique schema per worker
  - Run migrations for that schema
  - Provide a PrismaClient connected to that schema
  - Drop the schema after tests

### 2. **Example: test/prisma-fixture.ts**

```ts
import { test as base } from 'vitest';
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

export const test = base.extend<{ prisma: PrismaClient }>({
  prisma: [
    // eslint-disable-next-line no-empty-pattern
    async ({}, use) => {
      const workerId = process.env['VITEST_WORKER_ID'] || '0';
      const schema = `test_schema_${workerId}`;
      const baseUrl = process.env['DATABASE_URL'] || 'postgresql://postgres:postgres@localhost:5432/yourdb_test';
      const url = `${baseUrl}?schema=${schema}`;
      process.env['DATABASE_URL'] = url;

      // Create schema
      const tmpPrisma = new PrismaClient({ datasources: { db: { url } } });
      await tmpPrisma.$executeRawUnsafe(`CREATE SCHEMA IF NOT EXISTS "${schema}"`);
      await tmpPrisma.$disconnect();

      // Run migrations
      execSync('npx prisma migrate deploy', {
        stdio: 'inherit',
        env: { ...process.env, DATABASE_URL: url }
      });

      // Provide PrismaClient for tests
      const prisma = new PrismaClient({ datasources: { db: { url } } });
      await use(prisma);

      // Teardown: drop schema
      await prisma.$executeRawUnsafe(`DROP SCHEMA IF EXISTS "${schema}" CASCADE`);
      await prisma.$disconnect();
    },
    { scope: 'worker' }
  ]
});
```

### 3. **Import the Fixture in Your Test Files**

- Use the fixture in your test files to get an isolated PrismaClient:

```ts
import { test } from '@/test/prisma-fixture';

// Example test

test('creates a person', async ({ prisma }) => {
  await prisma.person.create({ data: { source_id: 'abc' } });
  // ... assertions ...
});
```

- This ensures each test worker uses its own schema, migrations are applied, and the schema is dropped after tests, enabling parallel, flake-free testing.

## dotenv-cli

- **Purpose:** Allows you to specify which `.env` file to use for a given command (e.g., `.env.test` for tests).
- **Install:**
  ```sh
  npm install dotenv-cli --save-dev
  ```
- **Usage in scripts:**
  ```json
  "db:migrate:test": "npx dotenv-cli -e .env.test -- npx prisma migrate deploy"
  ```

## Migration Commands (from package.json)

```
"db:migrate": "npx prisma migrate dev",
"db:migrate:test": "npx dotenv-cli -e .env.test -- npx prisma migrate deploy",
"db:migrate:reset": "npx prisma migrate reset --force",
"db:migrate:reset:test": "npx dotenv-cli -e .env.test -- npx prisma migrate reset --force"
```

- `db:migrate`: Run migrations on the development database.
- `db:migrate:test`: Run migrations on the test database using `.env.test`.
- `db:migrate:reset`: Reset the development database (drops and recreates, applies all migrations).
- `db:migrate:reset:test`: Reset the test database using `.env.test`.

## Best Practices

- **Always use the default Prisma Client output and import from `@prisma/client`.**
- **Keep separate `.env` files for dev and test.**
- **Use `dotenv-cli` to specify the environment file for test/CI scripts.**
- **Add a `postinstall` script to always generate Prisma Client after installing dependencies:**
  ```json
  "postinstall": "prisma generate"
  ``` 