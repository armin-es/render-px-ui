# @render-px/ui

Token-based React UI components — Button, Badge, Callout, InlineCode — built with [CVA](https://cva.style) and Tailwind CSS design tokens.

## Installation

```bash
npm install @render-px/ui
```

### Peer dependencies

```bash
npm install react react-dom
```

## Design tokens

Components rely on Tailwind CSS custom properties (design tokens) defined in your project. Add these to your Tailwind config or CSS:

| Token | Used by |
|---|---|
| `primary` | Button (primary/secondary variant), Badge (default), Callout title |
| `content` | Button (ghost/outline), Callout body text |
| `content-muted` | Badge (muted variant) |
| `content-border` | Button (outline border), Badge (muted bg) |
| `box-info-bg / border` | Badge (default bg), Callout (info) |
| `box-success-bg / border` | Badge (success), Callout (success) |
| `box-warning-bg / border` | Badge (warning), Callout (warning) |
| `box-yellow-bg / border` | Callout (note) |
| `inline-code-bg` | InlineCode background |

## Components

### Button

```tsx
import { Button } from '@render-px/ui'

<Button>Click me</Button>
<Button variant="secondary" size="lg">Secondary</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="outline" size="sm">Outline</Button>

// Polymorphic — render as a child element (e.g. a link)
<Button asChild>
  <a href="/page">Go to page</a>
</Button>
```

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'primary' \| 'secondary' \| 'ghost' \| 'outline'` | `'primary'` | Visual style |
| `size` | `'sm' \| 'md' \| 'lg'` | `'md'` | Padding and font size |
| `asChild` | `boolean` | `false` | Merge styles onto child element via Radix Slot |

Forwards all native `<button>` attributes and a `ref`.

---

### Badge

```tsx
import { Badge } from '@render-px/ui'

<Badge>Default</Badge>
<Badge variant="success">Success</Badge>
<Badge variant="warning" size="sm">Warning</Badge>
<Badge variant="muted" size="lg">Muted</Badge>
```

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'default' \| 'success' \| 'warning' \| 'muted'` | `'default'` | Color theme |
| `size` | `'sm' \| 'md' \| 'lg'` | `'md'` | Padding and font size |

Forwards all native `<span>` attributes.

---

### Callout

```tsx
import { Callout } from '@render-px/ui'

<Callout title="Info">This is an informational message.</Callout>
<Callout variant="success" title="Done">Your changes were saved.</Callout>
<Callout variant="warning" title="Heads up">This action is irreversible.</Callout>
<Callout variant="note" title="Note">Something worth knowing.</Callout>
```

| Prop | Type | Default | Description |
|---|---|---|---|
| `variant` | `'info' \| 'success' \| 'warning' \| 'note'` | `'info'` | Color theme for border, background, and title |
| `title` | `string` | — | Optional heading rendered above the body |

Forwards all native `<div>` attributes.

---

### InlineCode

```tsx
import { InlineCode } from '@render-px/ui'

<p>Run <InlineCode>npm install</InlineCode> to get started.</p>
```

Renders a styled `<code>` element. Forwards all native `<code>` attributes.

## Development

```bash
# Build once
npm run build

# Watch mode
npm run dev
```

Output is written to `dist/` in both ESM and CJS formats with TypeScript declarations.

## License

MIT
