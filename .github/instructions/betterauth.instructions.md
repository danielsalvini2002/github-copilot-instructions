
# Better Auth with Prisma & Next.js: AI Coding Standards

You are an expert Senior Software Engineer specializing in TypeScript, Next.js, and Security.
Your task is to generate code for an authentication system using Better Auth with Prisma ORM.

## 1. Core Technology Stack

* **Auth Library:** `better-auth` (Latest Version)
* **Database Adapter:** `better-auth/adapters/prisma`
* **Database:** PostgreSQL (via Prisma)
* **Framework:** Next.js (App Router)
* **Language:** TypeScript (Strict Mode)

## 2. File Structure & Conventions

Always strictly adhere to this file structure. Do not invent new paths.

* **Server Instance:** `src/lib/auth.ts`
* **Client Instance:** `src/lib/auth-client.ts`
* **Prisma Client:** `src/lib/prisma.ts` (Singleton pattern)
* **API Route:** `src/app/api/auth/[...all]/route.ts`
* **Middleware:** `src/proxy.ts`

## 3. Implementation Rules

### A. Database Schema (Prisma)

When generating `prisma/schema.prisma`, you MUST:

1.  Use `CUID` or `UUID` for all ID fields (e.g., `@default(cuid())`).
2.  Include `@@map("table_name")` to use snake_case table names in the DB (e.g., `@@map("user")`).
3.  Define `onDelete: Cascade` for relations (Session -> User, Account -> User).
4.  Create the four core models: `User`, `Session`, `Account`, `Verification`.

**Example User Model:**

```prisma
model User {
  id            String   @id @default(cuid())
  name          String
  email         String   @unique
  emailVerified Boolean
  image         String?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  sessions      Session[]
  accounts      Account[]

  @@map("user")
}

```

### B. Server-Side Configuration (`src/lib/auth.ts`)

1. **Always** import `prisma` from `@/lib/prisma` (singleton).
2. **Always** include the `nextCookies()` plugin in the `plugins` array. This is critical for Server Actions.
3. **Never** use `getCookie` manually inside the auth config; rely on the plugin.

```typescript
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { prisma } from "@/lib/prisma";
import { nextCookies } from "better-auth/next-js";

export const auth = betterAuth({
  database: prismaAdapter(prisma, { provider: "postgresql" }),
  emailAndPassword: { enabled: true },
  plugins: [nextCookies()] 
});

```

### C. Client-Side Configuration (`src/lib/auth-client.ts`)

1. Use `createAuthClient`.
2. Export destructured hooks: `signIn`, `signUp`, `useSession`, `signOut`.
3. Set `baseURL` to `process.env.NEXT_PUBLIC_APP_URL`.

### D. Authentication Patterns (Next.js)

1. **API Route:**
* Use `toNextJsHandler` in `src/app/api/auth/[...all]/route.ts`.


2. **Middleware (Optimistic Check):**
* Import `getSessionCookie` from `better-auth/cookies`.
* Use this ONLY for fast, insecure redirects (UX improvement).
* **DO NOT** perform database calls in middleware.


3. **Server Components (Secure Check):**
* Import `auth` from `@/lib/auth`.
* Import `headers` from `next/headers`.
* Call `await auth.api.getSession({ headers: await headers() })`.
* If null, redirect to `/sign-in`.



### E. Error Handling

When handling client-side errors (e.g., in `signIn`), use the `onError` callback.

* Display `ctx.error.message` to the user.
* Handle specific error codes if necessary (e.g., `ctx.error.status === 429` for rate limits).

## 4. Forbidden Patterns (DO NOT USE)

* [X] Do not use NextAuth.js or Auth.js syntax.
* [X] Do not create a new `PrismaClient()` instance inside `auth.ts`.
* [X] Do not use `pages/api` directory (unless explicitly requested).
* [X] Do not forget `nextCookies()` plugin when using App Router.

## 5. Tone & Style

* Generate code that is type-safe and strict.
* Prefer functional components and hooks.
* Add comments explaining why a security feature (like `headers()`) is used.