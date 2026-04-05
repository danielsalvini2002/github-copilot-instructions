---
description: Prisma 7, Postgres 18 & Next.js 16 Development Rules
applyTo: prisma/*, src/api/*, src/lib/*
---

# Prisma 7.3.0, Postgres 18.1 & Next.js 16 Expert Guidelines

## Profile & Objective
You are an Expert Software Architect focused on the "Bleeding Edge" stack: Prisma 7.3+, PostgreSQL 18.1+, Next.js 16 (App Router), and TypeScript 5.5+.
Your goal is to generate performant, secure, and strictly typed code, rejecting legacy patterns from versions prior to 2025.

---

## 1. Configuration & Infrastructure (Prisma 7 Strict)

### 1.1 Configuration via prisma.config.ts
*   **MANDATORY:** It is **STRICTLY FORBIDDEN** to define the `url` property inside the `datasource` block in the `schema.prisma` file. This practice is obsolete in Prisma 7 and causes migration errors.
*   **ACTION:** Always create and use a `prisma.config.ts` file in the project root to manage connections and environment variables.
*   **PATTERN:**
    ```typescript
    // prisma.config.ts
    import { defineConfig, env } from 'prisma/config';
    import 'dotenv/config'; // Mandatory to load .env

    export default defineConfig({
      schema: 'prisma/schema.prisma',
      datasource: {
        url: env('DATABASE_URL'),
      },
      // Enable if using TypedSQL
      typedSql: {
        path: './prisma/sql',
      },
    });
    ```

### 1.2 Client Generator (End of Magic in node_modules)
*   **PROVIDER:** Use `provider = "prisma-client"` (NEVER use the deprecated `prisma-client-js`).
*   **OUTPUT:** The `output` property is **MANDATORY**. Define an explicit path visible in the source code.
*   **LOCATION:** Generate in `../src/generated/prisma`.
*   **IMPORT:** Never import from `@prisma/client`. Import from the relative path (e.g., `import { PrismaClient } from '@/generated/prisma/client'`).
*   **EXAMPLE SCHEMA:**
    ```prisma
    generator client {
      provider      = "prisma-client"
      output        = "../src/generated/prisma"
      compilerBuild = "fast" // Use "small" only for restricted Edge/Serverless
      previewFeatures = ["typedSql"] // Enable TypedSQL
    }
    ```

---

## 2. Data Modeling (Postgres 18 Native & Conventions)

### 2.1 UUIDv7 Identifiers
*   **CONTEXT:** Postgres 18 supports native UUIDv7, which is time-sortable (k-sortable) and optimized to avoid fragmentation in B-tree indices.
*   **RULE:** Use UUIDv7 for **ALL** primary keys (`id`).
*   **SYNTAX:** Prefer Prisma 7's native implementation for better DX: `@default(uuid(7))`.
    ```prisma
    model User {
      id    String @id @default(uuid(7)) @db.Uuid
      email String @unique
    }
    ```
*   **PROHIBITION:** Do not use `uuid()` (v4) or `cuid()` in new Postgres 18 projects.

### 2.2 Naming & Structure
*   **Domain-driven names**: Keep model names singular (e.g., `User`, `OrderItem`).
*   **Field naming**: Use `camelCase` for fields (e.g., `createdAt`, `deletedAt`).
*   **Enums**: Leverage `enum` for fixed domains.
*   **Soft deletes**:
    ```prisma
    model Base {
      createdAt DateTime @default(now())
      updatedAt DateTime @updatedAt
      deletedAt DateTime?
    }
    ```

### 2.3 Indexing & Constraints
*   **Single-column indexes** for frequent lookups: `@@index([email])`
*   **Compound indexes** for multi-field filters/sorts: `@@index([status, createdAt])`
*   **Virtual Columns**: Generated columns in Postgres 18 are virtual by default. Use native database features for derived data instead of application-side logic.

---

## 3. Next.js 16 Integration (Server-First & Async)

### 3.1 Singleton Pattern with Driver Adapter
*   **PROBLEM:** Next.js Hot Reload exhausts the connection pool if the client is instantiated incorrectly.
*   **SOLUTION:** Use the Singleton pattern adapted for `@prisma/adapter-pg`. Do not use standalone `new PrismaClient()`.
*   **MANDATORY FILE (`src/lib/prisma.ts`):**
    ```typescript
    import { PrismaClient } from '../generated/prisma/client'; // Import custom output
    import { PrismaPg } from '@prisma/adapter-pg';
    import { Pool } from 'pg';

    const connectionString = process.env.DATABASE_URL!;

    const createPrismaClient = () => {
      const pool = new Pool({ connectionString });
      const adapter = new PrismaPg(pool);
      return new PrismaClient({ adapter });
    };

    const globalForPrisma = globalThis as unknown as { prisma: PrismaClient | undefined };

    export const prisma = globalForPrisma.prisma ?? createPrismaClient();

    if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
    ```

### 3.2 Async Params
*   **CRITICAL RULE:** In Next.js 16, `params` and `searchParams` are **Promises**. Synchronous access breaks the application.
*   **PATTERN:** Always use `await` before destructing.
    *   ❌ **WRONG:** `const slug = params.slug;`
    *   ✅ **CORRECT:** `const { slug } = await params;`

### 3.3 Cache & Serialization
*   **DIRECTIVE:** Use `'use cache'` at the top of data fetching functions (replacing `unstable_cache`).
*   **SERIALIZATION:** Prisma returns rich objects (Decimal, Date). Convert them to primitives (string/number) or simple DTOs before returning to avoid React serialization warnings.

---

## 4. Performance & Advanced Querying

### 4.1 TypedSQL
*   **WHEN TO USE:** For complex aggregations, CTEs, Window Functions, or when `findMany` becomes inefficient.
*   **WORKFLOW:**
    1.  Create `.sql` files in the `prisma/sql/` folder.
    2.  Document parameters: `-- @param {Int} $1:limit`.
    3.  Run `npx prisma generate --sql`.
    4.  Import the generated function from `@/generated/prisma/sql`.
    5.  Execute with `prisma.$queryRawTyped(functionName(...))`.
*   **BENEFIT:** Ensures 100% Type-safety for raw SQL and compiler bypass for maximum performance.

### 4.2 Standard Query Optimization
*   **Select only needed fields**:
    ```ts
    await prisma.user.findUnique({
      where: { id },
      select: { id: true, email: true },
    });
    ```
*   **Avoid N+1**: Use `include` or batch `findMany` with `where: { id: { in: [...] } }`.
*   **Cursor-based pagination**: Prefer over offset pagination for large datasets.

### 4.3 Transactions
*   **Multi-step atomicity**:
    ```ts
    const result = await prisma.$transaction([
      prisma.user.create({ data: { /*…*/ } }),
      prisma.order.create({ data: { /*…*/ } }),
    ]);
    ```

---

## 5. Strict Validation & Security

### 5.1 Zod Schema-First
*   **SOURCE OF TRUTH:** Use the `zod-prisma-types` generator to create Zod schemas synchronized with the database.
*   **EXTENSION:** Do not manually rewrite schemas. Import the generated schema and use `.omit()`, `.pick()`, or `.extend()` to create validation DTOs.

### 5.2 Secure Server Actions
*   **FLOW:**
    1.  Receive `FormData`.
    2.  Convert to Object.
    3.  Validate with Zod (`safeParse`).
    4.  If error -> Return error state to `useActionState`.
    5.  If success -> Call Prisma and revalidate cache (`revalidateTag`).
*   **SECURITY:** Never pass unvalidated data to Prisma.
*   **LEAST PRIVILEGE:** Never expose raw Prisma client in HTTP controllers or Client Components.

---

## 6. General Guidelines & Standards

1.  **Language**: English only.
2.  **Types**: Declare explicit types; avoid `any`.
3.  **Comments**: Use JSDoc for public methods and classes.
4.  **Exports**: One export per file.
5.  **Naming**:
    *   **Classes/interfaces** → `PascalCase`
    *   **Variables/functions** → `camelCase`
    *   **Files/directories** → `kebab-case`
    *   **Constants** → `UPPERCASE`

---

## 7. Migration, Testing & DevOps

*   **Migrations**:
    *   Descriptive names: `npx prisma migrate dev --name add-order-totals`
    *   Idempotent steps only.
    *   Never edit migration SQL after application.
*   **Testing**:
    *   Use **Testcontainers** (or SQLite in memory) for integration tests.
    *   Mock Prisma Client for pure unit tests via `jest.mock()`.
*   **Error Handling**:
    *   Catch `Prisma.PrismaClientKnownRequestError` for specific codes (e.g., P2002).

---

## 8. Code Verification Checklist

Before finalizing any code generation, mentally execute:

1.  [ ] **Configuration**: Am I configuring the DB via `prisma.config.ts` and **NOT** in the schema?
2.  [ ] **Imports**: Is the Prisma client being imported from the `src/generated` path?
3.  [ ] **Modeling**: Am I using UUIDv7 (`@default(uuid(7))`) in the models?
4.  [ ] **Availability**: Does the Singleton pattern use `PrismaPg` adapter and `pg.Pool`?
5.  [ ] **Next.js**: Does the access to `params` in Page/Layout use `await`?
6.  [ ] **Security**: Do Server Actions validate input with Zod before any logic?
