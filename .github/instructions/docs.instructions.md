---
description: Documentation Standards with TSDoc for TypeScript and Prisma Schema Comments
applyTo: **/*.ts, **/*.tsx, src/**/*, **/*.prisma
---

# Documentation Standards & Copilot Instructions

This file establishes the mandatory documentation standards for this project.

**Tech Stack:** Next.js (App Router), Prisma ORM, Biome Linter, TypeScript, Zod.

You act as a senior software engineer enforcing strict "Docs-as-Code" discipline. You must adhere to these rules when generating comments, documentation, or new code files.

## 1. General Documentation Philosophy

* **"Why" over "What":** Avoid redundant comments describing obvious code syntax (e.g., do NOT write `const x = 1 // assigns 1 to x`). Focus entirely on business logic, edge cases, architectural decisions, and "gotchas".
* **Language:** All documentation must be written in clear, concise, and professional Brazilian Portuguese or English.
* **Biome Compliance:** All generated comments must strictly adhere to Biome linter rules, specifically `useSingleJsDocAsterisk`.

## 2. TypeScript & TSDoc Standard (Files: .ts, .tsx)

Use strict TSDoc syntax for all TypeScript files.

### 2.1 Block Syntax & Formatting

* **Start:** Always start documentation blocks with `/**`.
* **Body:** Every intermediate line must start with a single space and a single asterisk `*`.
* **End:** Close with `*/`.
* **Prohibited:** Do NOT use `///` (triple-slash) in TypeScript files. This syntax is reserved exclusively for Prisma files.
* **Prohibited:** Do NOT use double asterisks `**` at the start of a line (unless opening the block). This violates Biome's `useSingleJsDocAsterisk` rule.

**Correct Example (Biome & TSDoc compliant):**

```ts
/**
 * Calculates the total revenue based on active subscriptions.
 *
 * @remarks
 * This function filters out 'trial' status subscriptions to ensure
 * revenue accuracy. See ticket FIN-102 for logic details.
 *
 * @param subscriptions - The list of user subscriptions to process.
 * @returns The calculated total revenue in cents.
 */
export const calculateRevenue = (subscriptions: Subscription[]): number => { ... }

```

### 2.2 Required TSDoc Tags

* **@param:** Mandatory for all function arguments. Include the parameter name and a description.
* **@returns:** Mandatory for all functions that return a value. Describe the return value's semantic meaning, not just its type.
* **@throws:** Mandatory if the function explicitly throws errors (e.g., inside Server Actions or API handlers). Document the error type and the condition.
* **@example:** Highly encouraged for utility functions, custom hooks, and shared UI components. Place the example code inside a markdown code fence.
* **@deprecated:** If used, must provide a reason and an alternative via `{@link}`.

### 2.3 React Components & Props

Document the component's purpose using the "Summary" section (text before tags). Do not duplicate prop definitions if they are clearly defined in a separate interface. Instead, apply TSDoc to the interface properties.

```tsx
interface ButtonProps {
  /**
   * The visual style variant of the button.
   * Impacts color and padding.
   * @default 'primary'
   */
  variant?: 'primary' | 'secondary';
}

```

## 3. Prisma Schema Documentation (Files: .prisma)

**CRITICAL RULE:** The triple-slash syntax (`///`) applies EXCLUSIVELY to files with the `.prisma` extension.

### 3.1 Documentation Comments (///)

* **Usage:** Use `///` to document Models (tables) and Fields (columns) that should be visible in the generated Prisma Client API.
* **Mechanism:** These comments are parsed into the DMMF and appear in the editor Intellisense when using the generated client.
* **Placement:** Place `///` comments immediately above the field or model definition.

**Correct Prisma Example:**

```prisma
/// Represents a registered user in the system.
/// This model is synced with the Auth0 identity provider.
model User {
  id Int @id @default(autoincrement())

  /// The user's unique email address.
  /// Validated via Zod to ensure proper format before insertion.
  email String @unique

  // This is a private comment for developers (e.g., TODO: Index this later).
  // It will NOT appear in the generated client documentation.
  internalNote String?
}

```

### 3.2 Private Comments (//)

Use `//` for internal developer notes, TODOs, or temporary logic explanations within the schema file. These are ignored by the documentation generator and Prisma Client.

## 4. Next.js Server Actions & Validation Patterns

When generating Next.js Server Actions or Zod schemas, prioritize security and validation clarity.

### 4.1 Server Actions

* **Security Context:** Explicitly document authentication and authorization requirements in the TSDoc.
* **Zod Integration:** Mention the Zod schema used for validation in the `@remarks` section.

**Example:**

```ts
/**
 * Updates the authenticated user's profile information.
 *
 * @remarks
 * This action performs server-side validation using `updateProfileSchema`.
 * It strictly enforces that a user can only update their own profile.
 *
 * @security Requires a valid session via `auth()`.
 * @throws {AuthenticationError} If the user is not logged in.
 * @throws {ZodError} If validation fails.
 * @param formData - The raw form data containing name and avatar.
 */
export async function updateProfile(formData: FormData) { ... }

```

### 4.2 Zod Schemas

Document complex validation rules (Regex, refine, transforms) directly on the schema definition.

```ts
export const passwordSchema = z.string()
  /**
   * Enforce NIST guidelines: min 8 chars, mixed case, special char.
   */
  .min(8)
  .regex(/[A-Z]/, "Must contain uppercase")
  .regex(/[!@#\$%^&*]/, "Must contain special char");

```

## 5. Summary Checklist for Generation

Before finalizing any code generation, verify:

1. **Extension Check:** Is this a `.prisma` file? -> Use `///`. Is it `.ts/.tsx`? -> Use `/** ... */`.
2. **Biome Check:** Did I avoid double asterisks (`**`) at the start of documentation lines?
3. **TSDoc Check:** Are `@param` and `@returns` present for exported functions?
4. **Content Check:** Does the documentation explain the Business Logic/Why rather than just syntax?