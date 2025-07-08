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