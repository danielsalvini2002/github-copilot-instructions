# GitHub Copilot Instructions for AV3M

You are an expert software engineer working on the AV3M project. Your task is to strictly adhere to the coding guidelines, architecture, and tech stack defined for this project.

## 🏗️ Project Context & Tech Stack

*   **Framework:** Next.js 16 (App Router, Server Actions, Turbopack)
*   **Language:** TypeScript 5.9+ (Strict Mode)
*   **UI Library:** Chakra UI v3 (Composed Components in `src/components/ui`)
*   **Database:** PostgreSQL with Prisma ORM v7
*   **Validation:** Zod
*   **Linting/Formatting:** Biome
*   **Icons:** Lucide React

## 📚 Instruction Reference

Detailed rules for each domain are located in the `.github/instructions/` folder. You **must** apply the concepts from the following files whenever generating related code:

1.  **Next.js & React Architecture**
    *   **File:** [`.github/instructions/next.instructions.md`](instructions/next.instructions.md)
    *   **Focus:** App Router patterns, Server vs Client Boundaries, Server Actions (`next-safe-action`), caching strategies, and file conventions.

2.  **UI & Design System (Chakra UI v3)**
    *   **File:** [`.github/instructions/chakraui.instructions.md`](instructions/chakraui.instructions.md)
    *   **Focus:** Using the new v3 root/trigger patterns, respecting `src/components/ui` snippet governance (vs `@chakra-ui/react`), theming with semantic tokens.
    *   **Docs:** Consult `AIGuides/chakraui_docs/llms-components.md` for specific component APIs.

3.  **Database & ORM (Prisma)**
    *   **File:** [`.github/instructions/prisma.instructions.md`](instructions/prisma.instructions.md)
    *   **Focus:** Schema definition, type-safe queries, transaction handling, and migration workflows.

4.  **Authentication (BetterAuth)**
    *   **File:** [`.github/instructions/betterauth.instructions.md`](instructions/betterauth.instructions.md)
    *   **Focus:** Authentication flows, session management, middleware protection, and integration with Next.js/Prisma.

5.  **TypeScript Standards**
    *   **File:** [`.github/instructions/typescript.instructions.md`](instructions/typescript.instructions.md)
    *   **Focus:** Strict typing, avoiding `any`, Interface Segregation, and type inference best practices.

6.  **Clean Code & Architecture**
    *   **File:** [`.github/instructions/cleancode.instructions.md`](instructions/cleancode.instructions.md)
    *   **Focus:** SOLID principles, AHA (Avoid Hasty Abstractions) instead of rigid DRY, Separation of Concerns, and Error Handling protocols.

7.  **Documentation Standards**
    *   **File:** [`.github/instructions/docs.instructions.md`](instructions/docs.instructions.md)
    *   **Focus:** TSDoc syntax, Prisma schema comments (`///` vs `//`), and documentation for Server Actions and Zod schemas.

## 🚀 KERNEL Principles Summary (Prompt Guide)

When interpreting requests, keep in mind the pillars of the KERNEL framework (defined in `AIGuides/prompt.md`):

*   **K (Keep it simple):** Direct and readable solutions over over-engineered ones.
*   **E (Easy to verify):** Generate code that is deterministic and easy to test.
*   **R (Reproducible results):** Strict adherence to the versions listed above (Next 16, Chakra v3, Prisma 7).
*   **N (Narrow scope):** Focus strictly on the requested task; do not sprawl into unrelated files.
*   **E (Explicit constraints):** Respect type constraints, Biome linting rules, and directory structures.
*   **L (Logical structure):** Organize code logically (Imports -> Types -> Components -> Logic).

---

**Important Note:** If an instruction in these files conflicts with your general knowledge, **prioritize these local project instructions**, as they reflect the specific configuration of the AV3M workspace.