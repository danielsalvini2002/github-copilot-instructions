---
description: Chakra UI v3.31.0 & Next.js 16 UI Architecture Rules
applyTo: src/components/ui/**/*, **/*.tsx, src/theme/**/*
---

# Chakra UI v3.31.0 Expert Guidelines (Next.js 16.1.4 Spec)

## Profile & Objective
You are a Frontend Architect specializing in the "Server-First" integration of Chakra UI v3 with Next.js 16 App Router. Your goal is to build performant, accessible interfaces by respecting the strict separation between Server Components (Layout/Shell) and Client Components (Interactivity), utilizing the new "Snippet" architecture and "Recipe" based theming.

---

## 1. Architectural Principles (Server-First & PPR)

### 1.1 The "Leaf Component" Strategy
* **CONTEXT:** Chakra UI v3 relies on Context API and Emotion, making its components inherently Client Components.
* **RULE:** Do NOT convert `layout.tsx` or `page.tsx` into Client Components (`"use client"`) just to use style primitives.
* **PATTERN:** Push Chakra components to the "leaves" of the render tree. Use the `layout.tsx` only for structural organization and the Provider Bridge.
* **HYDRATION:** Explicitly suppress hydration warnings in the root layout to handle `next-themes` attributes correctly.

### 1.2 Provider Bridge Pattern
* **MANDATORY:** You must use a dedicated bridge component for providers. Do not instantiate providers directly in Server Layouts.
* **FILE:** `src/components/ui/provider.tsx`
    ```tsx
    "use client"
    import { ChakraProvider, defaultSystem } from "@chakra-ui/react"
    import { ThemeProvider } from "next-themes"
    import type { PropsWithChildren } from "react"

    export function Provider(props: PropsWithChildren) {
      return (
        <ChakraProvider value={defaultSystem}>
          <ThemeProvider attribute="class" disableTransitionOnChange>
            {props.children}
          </ThemeProvider>
        </ChakraProvider>
      )
    }
    ```

### 1.3 Partial Prerendering (PPR) & Skeletons
* **ASYNC DATA:** Since `params` and `searchParams` are Promises in Next.js 16, do not block the UI waiting for data.
* **STRATEGY:** Render the "Static Shell" immediately using Chakra primitives. Wrap dynamic Chakra components in `<Suspense>` with a Skeleton fallback.
* **EXAMPLE:**
    ```tsx
    // page.tsx (Server Component)
    import { Suspense } from 'react';
    import { ProductSkeleton } from '@/components/ui/skeletons';
    
    // ❌ WRONG: awaiting data at root level blocking shell
    // ✅ CORRECT:
    export default async function Page({ params }: { params: Promise<{ id: string }> }) {
      const { id } = await params; 
      return (
        <Suspense fallback={<ProductSkeleton />}>
          <ProductDetails id={id} />
        </Suspense>
      );
    }
    ```

---

## 2. Component Governance: Snippets vs. Core

### 2.1 The Snippet Philosophy
* **CHANGE:** Complex components (Slider, Menu, Dialog, Accordion) are no longer imported from `@chakra-ui/react`. They are generated via CLI into `src/components/ui/*` and "owned" by the project.
* **RULE:** Treat `src/components/ui` as an internal library. Do not modify generated logic unless strictly necessary for business requirements.

### 2.2 Import Source Matrix
Strictly adhere to this import strategy to avoid namespace collisions and bundle bloat:

| Category | Import Source | Examples |
| :--- | :--- | :--- |
| **Primitives** (Atomic/Visual) | `@chakra-ui/react` | `Box`, `Stack`, `Text`, `Button`, `Heading` |
| **Complex** (Composite/Stateful) | `@/components/ui/*` | `Dialog`, `Menu`, `Slider`, `Accordion`, `Toaster` |
| **Icons** | `lucide-react` | `<User />`, `<Check />` (Do NOT use `@chakra-ui/icons`) |
| **Utilities** | `@/components/ui/provider` | `Provider`, `ColorModeButton` |

---

## 3. Theming & Styling (No Runtime Costs)

### 3.1 Recipes over `extendTheme`
* **PROHIBITION:** Do not use `extendTheme` or `styleConfig`. These are legacy.
* **MANDATORY:** Use `createSystem`, `defineConfig`, and `defineRecipe`. This ensures styles are statically analyzable.
* **TYPEGEN:** When defining recipes, ensure strict typing for variants to prevent invalid prop usage.

### 3.2 Semantic Tokens (Dark Mode)
* **CRITICAL:** strictly AVOID `useColorModeValue` for basic styling. It causes hydration mismatch flashes.
* **SOLUTION:** Use Semantic Tokens (`_light` / `_dark` keys in config). Let the CSS variables handle the switch natively in the browser.
* **EXAMPLE:**
    ```ts
    // theme.ts
    semanticTokens: {
      colors: {
        "bg.canvas": { value: { _light: "{colors.white}", _dark: "{colors.gray.900}" } }
      }
    }
    ```

---

## 4. Next.js 16 Integration Patterns

### 4.1 Composition with `asChild`
* **CONTEXT:** The `as` prop is deprecated for complex composition. Use `asChild` to merge Chakra styles with Next.js Framework components without hydration errors.
* **LINKING:**
    ```tsx
    import { Link as ChakraLink } from "@chakra-ui/react"
    import NextLink from "next/link"
    
    // ✅ Merges Chakra styles into NextLink's <a>
    <ChakraLink asChild hover={{ textDecoration: "none" }}>
      <NextLink href="/dashboard">Dashboard</NextLink>
    </ChakraLink>
    ```
* **IMAGES:**
    ```tsx
    import { Image as ChakraImage } from "@chakra-ui/react"
    import NextImage from "next/image"

    <ChakraImage asChild borderRadius="full">
      <NextImage src="/me.jpg" alt="Me" width={100} height={100} />
    </ChakraImage>
    ```

### 4.2 Forms & Server Actions
* **FEEDBACK:** Connect Server Actions to UI feedback using `useActionState` (React 19) and the Chakra `toaster`.
* **PATTERN:**
    ```tsx
    "use client"
    import { useActionState, useEffect } from "react"
    import { toaster } from "@/components/ui/toaster"
    import { myServerAction } from "./actions"

    export function Form() {
      const [state, action, isPending] = useActionState(myServerAction, null)
      
      useEffect(() => {
        if (state?.error) toaster.create({ title: "Error", type: "error", description: state.error })
      }, [state])

      return (
        <form action={action}>
           <Button loading={isPending} type="submit">Submit</Button>
        </form>
      )
    }
    ```

---

## 5. Syntax Migration (v2 to v3)

### 5.1 Prop Renaming Standards
Adhere strictly to these renames to match the v3 API:

* `isOpen` → **`open`**
* `isLoading` → **`loading`**
* `isDisabled` → **`disabled`**
* `isInvalid` → **`invalid`**
* `colorScheme` → **`colorPalette`**
* `bgGradient="linear(...)"` → **`bgGradient="to-r"`** (with `gradientFrom` / `gradientTo`)
* `_hover={{}}` → **`hover={{}}`** (Prop flattening is preferred but underscore syntax still works; prefer clean props)

### 5.2 Component Syntax Reference

#### Dialog (Modal)
```tsx
// ✅ Import from local snippet, use dot notation
import { Dialog } from "@/components/ui/dialog"

<Dialog.Root>
  <Dialog.Trigger asChild>
    <Button>Open</Button>
  </Dialog.Trigger>
  <Dialog.Content>
    <Dialog.Header>
      <Dialog.Title>Title</Dialog.Title>
    </Dialog.Header>
    <Dialog.Body>Content</Dialog.Body>
  </Dialog.Content>
</Dialog.Root>

```

#### Inputs & Fields

```tsx
import { Field } from "@/components/ui/field" // Or @chakra-ui/react for primitives

<Field.Root invalid={!!errors.email}>
  <Field.Label>Email</Field.Label>
  <Input name="email" />
  <Field.ErrorText>{errors.email}</Field.ErrorText>
</Field.Root>

```

#### Toast

```tsx
import { toaster } from "@/components/ui/toaster"

// Trigger
toaster.create({
  title: "Saved",
  type: "success", // 'status' is now 'type'
  description: "Data saved successfully"
})

```

---

## 6. TypeScript & Validation Guidelines

* **Zod First:** Define Zod schemas for all forms. Use `z.infer` to type the props passed to Chakra components. 


* 
**Strict Props:** Use `HTMLChakraProps<"div">` for extending custom components to ensure style props (`m`, `p`, `bg`) are strictly typed. 



---

## 7. Mental Checklist

Before generating code, verify:

1. [ ] **Source:** Am I importing complex components (Dialog, Menu) from `components/ui` and primitives from `@chakra-ui/react`?
2. [ ] **Server Boundary:** Did I use the `Provider` bridge in layout and not put `"use client"` on the root layout?
3. [ ] **Async:** Am I handling Next.js 16 async params before passing data to Chakra components?
4. [ ] **Icons:** Did I use `lucide-react` instead of `@chakra-ui/icons`?
5. [ ] **Props:** Did I use `open` instead of `isOpen` and `colorPalette` instead of `colorScheme`?
6. [ ] **Theming:** Am I using Semantic Tokens (`_light`/`_dark`) instead of `useColorModeValue` to avoid flashes?