# TypeScript Koa API Starter

This is a minimal professional TypeScript Koa application, set up with:

- TypeScript strict mode
- Koa + koa-router
- Vitest for unit + HTTP testing (with coverage)
- ESLint using the modern flat config style
- Prettier for consistent formatting
- Husky + lint-staged for pre-commit checks
- dotenv for environment configuration

It’s a clean engineering baseline ready for adding real business logic.

## 🚀 Setup Instructions

### 1. Initialize the project

```bash
mkdir my-api
cd my-api
npm init -y
```

### 2. Install TypeScript & Node tooling

```bash
npm install -D typescript ts-node ts-node-dev @types/node
```

Generate a `tsconfig.json`:

```bash
npx tsc --init
```

Edit to include:

```json
{
  "compilerOptions": {
    "target": "es2019",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

### 3. Install Koa & router

```bash
npm install koa koa-router
npm install -D @types/koa @types/koa-router
```

### 4. Setup project structure and minimal app

```bash
mkdir -p src/routes
touch src/index.ts src/routes/index.ts
```

**src/index.ts**

```typescript
import Koa from "koa";
import indexRoutes from "./routes/index";
import dotenv from "dotenv";

dotenv.config();

const app = new Koa();
const PORT = process.env.PORT || 3000;

app
  .use(indexRoutes.routes())
  .use(indexRoutes.allowedMethods());

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

**src/routes/index.ts**

```typescript
import Router from "koa-router";

const router = new Router();

router.get("/", async ctx => {
  ctx.body = "Hello World from separate route file!";
});

export default router;
```

### 5. Configure dotenv

```bash
npm install dotenv
```

Create a `.env`:

```bash
PORT=3000
```

### 6. Setup ESLint (modern flat config)

```bash
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin @eslint/js eslint-config-prettier globals
npx eslint --init
```

Then update `eslint.config.mjs`:

```javascript
import js from "@eslint/js";
import globals from "globals";
import tseslint from "typescript-eslint";
import prettier from "eslint-config-prettier";
import { defineConfig } from "eslint/config";

export default defineConfig([
  {
    files: ["**/*.{js,mjs,cjs,ts,mts,cts}"],
    plugins: { js },
    extends: ["js/recommended", prettier]
  },
  {
    files: ["**/*.js"],
    languageOptions: { sourceType: "commonjs" }
  },
  {
    files: ["**/*.{js,mjs,cjs,ts,mts,cts}"],
    languageOptions: { globals: globals.node }
  },
  tseslint.configs.recommended
]);
```

### 7. Setup Prettier

```bash
npm install -D prettier
```

Create `.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "trailingComma": "es5",
  "arrowParens": "avoid"
}
```

Also `.editorconfig`:

```
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
```

### 8. Add Vitest + Supertest for testing

```bash
npm install -D vitest @vitest/coverage-v8 supertest
```

Create `vitest.config.ts`:

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "node",
    coverage: {
      provider: "v8"
    }
  }
});
```

Create `src/routes/index.test.ts`:

```typescript
import { describe, it, expect } from "vitest";
import request from "supertest";
import Koa from "koa";
import indexRoutes from "./index";

describe("indexRoutes", () => {
  it("GET / returns Hello World", async () => {
    const app = new Koa();
    app.use(indexRoutes.routes()).use(indexRoutes.allowedMethods());

    const response = await request(app.callback()).get("/");
    expect(response.status).toBe(200);
    expect(response.text).toBe("Hello World from separate route file!");
  });
});
```

### 9. Setup Husky + lint-staged

```bash
npm install -D husky lint-staged
npx husky init
```

Update `.husky/pre-commit`:

```bash
npx lint-staged --allow-empty
npm run test
```

Add lint-staged config to `package.json`:

```json
"lint-staged": {
  "*.{ts,js}": ["eslint --fix", "prettier --write"]
}
```

### 10. Add npm scripts

```json
"scripts": {
  "dev": "ts-node-dev --respawn --transpile-only src/index.ts",
  "build": "tsc",
  "start": "node dist/index.js",
  "lint": "eslint src --ext .ts",
  "format": "prettier --write .",
  "typecheck": "tsc --noEmit",
  "test": "vitest run",
  "test:coverage": "vitest run --coverage",
  "prepare": "husky"
}
```

---

## ✅ Complete

You now have a fully type-checked, linted, formatted, tested Koa TypeScript API:

- Strict TS + lint + prettier enforced on every commit with Husky + lint-staged
- Tests + coverage with Vitest and Supertest
- Ready to expand with more routes, middleware, or DB.