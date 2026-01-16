---
name: base-ui-best-practices
description: Base UI implementation guidelines for building accessible, unstyled React components. Use this skill when writing, reviewing, or implementing Base UI components to ensure proper patterns and accessibility. Triggers on tasks involving Base UI components like Dialog, Menu, Popover, Select, Tabs, etc.
---

# Base UI Best Practices

## Overview

Base UI is an unstyled UI component library for building accessible user interfaces, from the creators of Radix, Floating UI, and Material UI. It provides headless components that you can style with any CSS solution.

## When to Apply

Reference these guidelines when:
- Building UI components with Base UI
- Implementing accessible modals, menus, popovers, or forms
- Creating custom styled components on top of Base UI primitives
- Reviewing code for accessibility and composition patterns
- Migrating from Radix UI or other headless libraries

## Key Principles

| Principle | Description |
|-----------|-------------|
| Headless/Unstyled | No default styles - full control over CSS |
| Composable | Build complex UIs from primitive parts |
| Accessible | WAI-ARIA compliant out of the box |
| Flexible | Works with any styling solution |

## Quick Start Setup

### Installation

```bash
npm install @base-ui/react
```

All components are in a single package. Base UI is tree-shakable.

### Portal Setup (Required)

Base UI uses portals for popups (Dialog, Popover, Menu, etc.). Add this to ensure popups appear above page content:

```tsx
// layout.tsx
<body>
  <div className="root">
    {children}
  </div>
</body>
```

```css
/* styles.css */
.root {
  isolation: isolate;
}
```

### iOS 26+ Safari Fix

For backdrops to cover the full viewport on iOS 26+:

```css
body {
  position: relative;
}
```

## Accessibility

Base UI handles complex accessibility automatically:

### Built-in Features
- **ARIA attributes** - Roles, labels, and relationships
- **Keyboard navigation** - Arrow keys, Tab, Escape, Enter, Home, End
- **Focus management** - Automatic focus trapping in modals
- **Screen reader support** - Live regions for dynamic content

### Developer Responsibilities

**Focus indicators** - Style `:focus-visible` for keyboard users:
```css
.button:focus-visible {
  outline: 2px solid var(--color-blue);
  outline-offset: 2px;
}
```

**Color contrast** - Meet WCAG requirements (consider [APCA](https://apcacontrast.com/))

**Accessible labels** - All form controls need labels:
```tsx
// Option 1: Wrap in label
<label>
  Email
  <Input />
</label>

// Option 2: Use Field component (recommended)
<Field.Root>
  <Field.Label>Email</Field.Label>
  <Input />
</Field.Root>

// Option 3: aria-label for icon buttons
<Button aria-label="Close">
  <CloseIcon />
</Button>
```

## Available Components

### Overlays & Modals
- **Dialog** - Modal windows for focused interactions
- **Alert Dialog** - Modal requiring user acknowledgment
- **Popover** - Floating content anchored to triggers
- **Tooltip** - Contextual information on hover/focus
- **Toast** - Temporary notifications

### Navigation & Menus
- **Menu** - Dropdown menus with keyboard navigation
- **Menubar** - Horizontal menu bar
- **Context Menu** - Right-click menus
- **Navigation Menu** - Site navigation with submenus
- **Tabs** - Tabbed content organization

### Form Controls
- **Button** - Clickable button element
- **Checkbox** - Single checkbox input
- **Checkbox Group** - Multiple related checkboxes
- **Radio** - Radio button group
- **Switch** - Toggle switch
- **Toggle** - Pressable toggle button
- **Toggle Group** - Group of toggle buttons
- **Input** - Text input field
- **Number Field** - Numeric input with increment/decrement
- **Select** - Dropdown selection
- **Combobox** - Searchable dropdown
- **Autocomplete** - Input with suggestions
- **Slider** - Range slider input
- **Field** - Form field wrapper with label/error
- **Fieldset** - Group of form fields
- **Form** - Form container with validation

### Layout & Display
- **Accordion** - Expandable/collapsible sections
- **Collapsible** - Single collapsible section
- **Scroll Area** - Custom scrollable container
- **Separator** - Visual divider
- **Toolbar** - Grouped action buttons
- **Avatar** - User avatar display
- **Meter** - Visual meter/gauge
- **Progress** - Progress indicator
- **Preview Card** - Hoverable preview content

### Utilities
- **CSP Provider** - Content Security Policy support
- **Direction Provider** - RTL/LTR support
- **mergeProps** - Utility for merging props
- **useRender** - Custom render hook

## Quick Reference

### Component Composition Pattern

Base UI components use a parts-based composition pattern:

```tsx
import { Dialog } from '@base-ui/react/dialog';

function MyDialog() {
  return (
    <Dialog.Root>
      <Dialog.Trigger>Open</Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Backdrop className="backdrop" />
        <Dialog.Popup className="popup">
          <Dialog.Title>Title</Dialog.Title>
          <Dialog.Description>Description</Dialog.Description>
          <Dialog.Close>Close</Dialog.Close>
        </Dialog.Popup>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

### Controlled vs Uncontrolled

```tsx
// Uncontrolled (internal state)
<Dialog.Root defaultOpen={false}>

// Controlled (external state)
const [open, setOpen] = useState(false);
<Dialog.Root open={open} onOpenChange={setOpen}>
```

### Styling Approaches

```tsx
// CSS classes
<Dialog.Popup className="my-dialog">

// CSS-in-JS / Tailwind
<Dialog.Popup className="bg-white rounded-lg shadow-xl p-6">

// Render prop for state-based styling
<Dialog.Popup className={(state) => state.open ? 'visible' : 'hidden'}>
```

## The `render` Prop (Important)

**Base UI uses a `render` prop instead of Radix's `asChild`.** This is the primary way to change the rendered element or access component state.

### Changing the Rendered Element

```tsx
// Default: renders as <button>
<Dialog.Trigger>Open</Dialog.Trigger>

// Override: render as <a> instead
<Dialog.Trigger render={<a href="#dialog" />}>Open</Dialog.Trigger>

// Override: render as custom component
<Dialog.Trigger render={<MyButton />}>Open</Dialog.Trigger>
```

### Accessing State with Render Callback

The callback form gives you access to props and internal state:

```tsx
<Dialog.Popup
  render={(props, state) => (
    <div {...props} data-nested={state.nested}>
      {props.children}
    </div>
  )}
/>

<Menu.Item
  render={(props, state) => (
    <a {...props} className={state.highlighted ? 'highlighted' : ''}>
      {props.children}
    </a>
  )}
/>
```

### Building Custom Components with `useRender`

Use the `useRender` hook to add render prop support to your own components:

```tsx
import { useRender } from '@base-ui/react/use-render';
import { mergeProps } from '@base-ui/react/merge-props';

interface ButtonProps extends useRender.ComponentProps<'button'> {}

function Button({ render, ...props }: ButtonProps) {
  return useRender({
    defaultTagName: 'button',
    render,
    props: mergeProps<'button'>({ className: 'my-button' }, props),
  });
}

// Usage
<Button>Click me</Button>                        // renders <button>
<Button render={<a href="/home" />}>Home</Button> // renders <a>
```

### Migrating from Radix `asChild`

```tsx
// Radix UI (asChild)
<Button asChild>
  <a href="/contact">Contact</a>
</Button>

// Base UI (render prop)
<Button render={<a href="/contact" />}>Contact</Button>
```

**Full documentation:** `references/utils/use-render.md`

## Handbook Guides

**IMPORTANT:** These in-depth guides cover essential patterns. Reference them for specific implementation needs.

| Guide | When to Use | File |
|-------|-------------|------|
| **Styling** | Applying styles with Tailwind, CSS Modules, CSS-in-JS. Using `className` functions, data attributes, CSS variables for state-based styling. | `references/handbook/styling.md` |
| **Animation** | Adding enter/exit animations, CSS transitions, using Motion library, animating popups and dialogs on mount/unmount. | `references/handbook/animation.md` |
| **Composition** | Wrapping Base UI parts in custom components, using `render` prop to change rendered elements, building reusable component abstractions. | `references/handbook/composition.md` |
| **Customization** | Handling events, preventing default behaviors, controlling internal state, intercepting Base UI event handlers. | `references/handbook/customization.md` |
| **Forms** | Building forms with Field/Fieldset/Form, validation (constraint & custom), displaying errors, integrating React Hook Form or TanStack Form. | `references/handbook/forms.md` |
| **TypeScript** | Accessing component types via namespaces (`Dialog.Props`, `Dialog.State`), typing custom wrappers, event types. | `references/handbook/typescript.md` |

## References

Full documentation with code examples:

- `references/base-ui-guidelines.md` - Complete implementation guide
- `references/components/` - Individual component documentation (35 components)
- `references/handbook/` - In-depth guides (see table above)
- `references/utils/` - Utility functions:
  - `use-render.md` - Hook for enabling render prop in custom components
  - `merge-props.md` - Safely merge event handlers, classNames, and styles

To look up a specific component:
```
grep -l "Dialog" references/components/
grep -l "Menu" references/components/
grep -l "Popover" references/components/
```
