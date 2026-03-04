# Architecture: @render-px/ui

A token-based React component library. Components contain no hardcoded colors or
spacing values. Every visual property resolves through a two-layer indirection:
CSS custom properties define the design intent, and Tailwind's `theme.extend`
bridges those variables into utility classes that components consume.

This makes components theme-agnostic by construction. Light/dark mode, brand
changes, or per-product themes are purely a CSS variable concern - no component
code changes required.

---

## Layer 1: Design Tokens (CSS Custom Properties)

Define tokens in your global stylesheet under `@layer base`. Values use raw HSL
channels (no `hsl()` wrapper), which enables Tailwind's opacity modifier syntax
(`bg-primary/20`).

**Token groups:**

| Group | Variables | Purpose |
|---|---|---|
| Layout | `--sidebar-bg`, `--header-bg`, `--content-bg` | Surface backgrounds |
| Typography | `--content-text`, `--content-text-muted` | Text hierarchy |
| Structure | `--content-border`, `--card-bg`, `--preview-bg` | Borders and containers |
| Semantic color | `--box-info-*`, `--box-success-*`, `--box-warning-*`, `--box-yellow-*` | Status/feedback |
| Inline | `--inline-code-bg` | Code-specific surface |
| Brand | `--link` | Single primary accent |

Example for light and dark themes:

```css
@layer base {
  :root, .theme-light {
    --link: 317 77% 55%;
    --content-text: 0 0% 9%;
    --content-text-muted: 0 0% 45%;
    --content-border: 0 0% 90%;
    --inline-code-bg: 0 0% 93%;
    --box-info-bg: 317 77% 55% / 0.08;
    --box-info-border: 317 77% 55% / 0.35;
    --box-success-bg: 142 76% 36% / 0.1;
    --box-success-border: 142 76% 36% / 0.4;
    --box-warning-bg: 38 92% 50% / 0.1;
    --box-warning-border: 38 92% 50% / 0.4;
    --box-yellow-bg: 48 96% 53% / 0.12;
    --box-yellow-border: 48 96% 53% / 0.4;
  }

  @media (prefers-color-scheme: dark) {
    :root:not(.theme-light) {
      --link: 317 77% 55%;
      --content-text: 0 0% 95%;
      --content-text-muted: 0 0% 55%;
      --content-border: 0 0% 22%;
      --inline-code-bg: 0 0% 20%;
      --box-info-bg: 317 77% 55% / 0.15;
      --box-info-border: 317 77% 55% / 0.4;
      --box-success-bg: 142 76% 36% / 0.2;
      --box-success-border: 142 76% 36% / 0.5;
      --box-warning-bg: 38 92% 50% / 0.2;
      --box-warning-border: 38 92% 50% / 0.5;
      --box-yellow-bg: 48 96% 53% / 0.15;
      --box-yellow-border: 48 96% 53% / 0.45;
    }
  }

  .theme-dark {
    /* same values as the media query block above */
  }
}
```

The three-selector pattern supports three modes without JavaScript: auto (OS),
forced light, forced dark.

---

## Layer 2: Tailwind Bridge (`tailwind.config.js`)

`theme.extend.colors` maps every CSS variable into a named Tailwind token:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './src/**/*.{ts,tsx}',
    // Required: include the package so Tailwind does not purge its classes
    './node_modules/@render-px/ui/dist/**/*.{js,mjs}',
  ],
  theme: {
    extend: {
      colors: {
        primary: 'hsl(var(--link))',
        content: {
          DEFAULT: 'hsl(var(--content-text))',
          muted:   'hsl(var(--content-text-muted))',
          border:  'hsl(var(--content-border))',
        },
        'inline-code': {
          bg: 'hsl(var(--inline-code-bg))',
        },
        'box-info': {
          bg:     'hsl(var(--box-info-bg))',
          border: 'hsl(var(--box-info-border))',
        },
        'box-success': {
          bg:     'hsl(var(--box-success-bg))',
          border: 'hsl(var(--box-success-border))',
        },
        'box-warning': {
          bg:     'hsl(var(--box-warning-bg))',
          border: 'hsl(var(--box-warning-border))',
        },
        'box-yellow': {
          bg:     'hsl(var(--box-yellow-bg))',
          border: 'hsl(var(--box-yellow-border))',
        },
      },
    },
  },
}
```

Because the raw HSL values omit the `hsl()` call, the variable stays composable.
`bg-primary/10` works correctly; `bg-primary` also works.

---

## Layer 3: Components

All four components share the same structural pattern.

### Variant management: `class-variance-authority` (CVA)

CVA provides TypeScript-safe variant APIs. The `cva()` call defines a base class
string plus variant maps. TypeScript infers the allowed values directly from the
object keys - passing an invalid variant is a compile error.

```ts
const buttonVariants = cva(
  'inline-flex items-center ...',  // base classes
  {
    variants: {
      variant: {
        primary:   'bg-primary text-white hover:opacity-90',
        secondary: 'border-2 border-primary text-primary hover:opacity-90',
        ghost:     'text-content hover:bg-content-border/20',
        outline:   'border border-content-border text-content hover:bg-content-border/20',
      },
      size: {
        sm: 'px-3 py-1.5 text-sm rounded-md',
        md: 'px-4 py-2 text-sm rounded-lg',
        lg: 'px-6 py-3 text-base rounded-lg',
      },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  }
)
```

### Polymorphic rendering: Radix `Slot` (`asChild`)

`Button` exposes `asChild?: boolean`. When true, `Slot` merges all props
(including the CVA `className` output) onto the direct child element instead of
rendering a `<button>`. This eliminates wrapper nodes while preserving all styles:

```tsx
const Comp = asChild ? Slot : 'button'
return <Comp className={buttonVariants({ variant, size, className })} ref={ref} {...props} />
```

### Ref forwarding

`Button` uses `forwardRef` for focus management and imperative access. The other
three components forward refs implicitly via prop spreading onto native elements.

### Token dependency map

| Component | Tokens consumed |
|---|---|
| Button | `primary`, `content-border`, `content` |
| Badge | `box-info-bg`, `box-success-bg`, `box-warning-bg`, `content-border` |
| Callout | `box-{info,success,warning,yellow}-{bg,border}`, `content`, `primary` |
| InlineCode | `inline-code-bg` |

---

## Package Structure

```
@render-px/ui
├── src/
│   ├── index.ts                  barrel: exports all components + types
│   └── components/
│       ├── button.tsx
│       ├── badge.tsx
│       ├── callout.tsx
│       └── inline-code.tsx
├── dist/
│   ├── index.js                  CJS (require)
│   ├── index.mjs                 ESM (import)
│   └── index.d.ts                TypeScript declarations
├── tsup.config.ts                builds CJS + ESM + .d.ts, externalizes react
└── tsconfig.json                 ESNext, bundler resolution, react-jsx
```

`react` and `react-dom` are `peerDependencies`. The package has two runtime
dependencies: `class-variance-authority` and `@radix-ui/react-slot`.

---

## Consumer Setup Checklist

1. **Install the package**
   ```
   npm install @render-px/ui
   ```

2. **Define CSS variables** in your global stylesheet (see Layer 1 above).

3. **Register Tailwind tokens** in `tailwind.config.js` (see Layer 2 above).

4. **Add the package to Tailwind's `content` scan** so purgecss does not strip
   classes used inside `node_modules/@render-px/ui/dist`.

---

## Design Decisions

**Why CSS custom properties instead of a JS theme object?**
CSS variables cascade naturally through the DOM and respond to media queries and
class toggles without any JavaScript. The three-mode theme system (auto/light/dark)
requires zero JS to switch.

**Why CVA instead of inline conditionals?**
CVA keeps variant logic centralized and the TypeScript types derive from the
variant definition automatically. Adding a new variant is one entry in one object,
immediately type-safe.

**Why a standalone npm package?**
The components were designed and validated inside the portfolio app, then extracted
once stable. The portfolio continues to use local copies during development, while
external consumers install from the registry. The component code is identical in
both places.
