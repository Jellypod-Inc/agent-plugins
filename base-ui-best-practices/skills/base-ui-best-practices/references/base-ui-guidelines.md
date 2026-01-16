# Base UI Guidelines

**Version 1.0.0**
January 2026

> **Note:**
> This document is for agents and LLMs to follow when building,
> maintaining, or refactoring Base UI components. Humans may also
> find it useful, but guidance here is optimized for AI-assisted workflows.

---

## Abstract

Base UI is an unstyled, accessible React component library from the creators of Radix, Floating UI, and Material UI. This guide covers patterns, best practices, and implementation details for building production-quality UIs with Base UI.

---

## Table of Contents

1. [Installation & Setup](#1-installation--setup)
2. [Component Composition Pattern](#2-component-composition-pattern)
3. [State Management](#3-state-management)
4. [Styling Approaches](#4-styling-approaches)
5. [Accessibility](#5-accessibility)
6. [Common Patterns](#6-common-patterns)
7. [Component Reference](#7-component-reference)

---

## 1. Installation & Setup

### Package Installation

```bash
npm install @base-ui/react
# or
yarn add @base-ui/react
# or
pnpm add @base-ui/react
```

### Import Pattern

Base UI uses path-based imports for tree-shaking:

```tsx
// Correct: Import from specific component path
import { Dialog } from '@base-ui/react/dialog';
import { Menu } from '@base-ui/react/menu';
import { Select } from '@base-ui/react/select';

// Avoid: Barrel imports (may increase bundle size)
import { Dialog, Menu, Select } from '@base-ui/react';
```

---

## 2. Component Composition Pattern

Base UI components use a **parts-based composition pattern**. Each component is split into semantic parts that you compose together.

### Anatomy Example

```tsx
import { Dialog } from '@base-ui/react/dialog';

function MyDialog() {
  return (
    <Dialog.Root>
      {/* Trigger opens the dialog */}
      <Dialog.Trigger>Open</Dialog.Trigger>

      {/* Portal renders content outside the DOM hierarchy */}
      <Dialog.Portal>
        {/* Backdrop is the overlay behind the dialog */}
        <Dialog.Backdrop className="backdrop" />

        {/* Popup is the main dialog container */}
        <Dialog.Popup className="popup">
          {/* Title provides accessible labeling */}
          <Dialog.Title>Dialog Title</Dialog.Title>

          {/* Description provides additional context */}
          <Dialog.Description>
            Dialog description text.
          </Dialog.Description>

          {/* Close dismisses the dialog */}
          <Dialog.Close>Close</Dialog.Close>
        </Dialog.Popup>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

### Common Component Parts

| Part | Purpose | Renders |
|------|---------|---------|
| `Root` | Container managing state | No element (fragment) |
| `Trigger` | Opens/activates the component | `<button>` |
| `Portal` | Moves content to document body | `<div>` |
| `Backdrop` | Overlay behind floating content | `<div>` |
| `Positioner` | Handles positioning logic | `<div>` |
| `Popup` | Main content container | `<div>` |
| `Title` | Accessible heading | `<h2>` |
| `Description` | Accessible description | `<p>` |
| `Close` | Dismisses the component | `<button>` |
| `Arrow` | Directional indicator | `<div>` |

---

## 3. State Management

### Uncontrolled (Default)

Let the component manage its own state:

```tsx
// Uncontrolled - component manages open state internally
<Dialog.Root defaultOpen={false}>
  <Dialog.Trigger>Open</Dialog.Trigger>
  {/* ... */}
</Dialog.Root>
```

### Controlled

Manage state externally for full control:

```tsx
const [open, setOpen] = useState(false);

// Controlled - you manage the state
<Dialog.Root open={open} onOpenChange={setOpen}>
  <Dialog.Trigger>Open</Dialog.Trigger>
  {/* ... */}
</Dialog.Root>
```

### When to Use Each

| Pattern | Use When |
|---------|----------|
| Uncontrolled | Simple use cases, no external state needed |
| Controlled | Need to sync with other state, programmatic control, validation |

---

## 4. Styling Approaches

Base UI is completely unstyled. Choose your preferred styling method:

### CSS Classes

```tsx
<Dialog.Popup className="my-dialog-popup">
```

### Tailwind CSS

```tsx
<Dialog.Popup className="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white rounded-lg shadow-xl p-6">
```

### CSS Modules

```tsx
import styles from './Dialog.module.css';

<Dialog.Popup className={styles.popup}>
```

### State-Based Styling

Components accept functions for dynamic styling based on state:

```tsx
<Dialog.Popup
  className={(state) =>
    state.open ? 'dialog-open' : 'dialog-closed'
  }
>

<Dialog.Popup
  style={(state) => ({
    opacity: state.open ? 1 : 0,
    transform: state.open ? 'scale(1)' : 'scale(0.95)',
  })}
>
```

### Data Attributes

Components expose data attributes for CSS styling:

```css
/* Style based on state */
.dialog-popup[data-open] {
  opacity: 1;
}

.dialog-popup[data-closed] {
  opacity: 0;
}

/* Animation states */
.dialog-popup[data-starting-style] {
  opacity: 0;
  transform: scale(0.95);
}

.dialog-popup[data-ending-style] {
  opacity: 0;
  transform: scale(0.95);
}
```

### Common Data Attributes

| Attribute | Description |
|-----------|-------------|
| `data-open` | Present when component is open |
| `data-closed` | Present when component is closed |
| `data-starting-style` | Present during enter animation |
| `data-ending-style` | Present during exit animation |
| `data-disabled` | Present when disabled |
| `data-highlighted` | Present when item is highlighted (menus) |
| `data-active` | Present when active (tabs, toggles) |

---

## 5. Accessibility

Base UI components are WAI-ARIA compliant out of the box.

### Automatic Accessibility Features

- **Focus management**: Proper focus trapping in modals
- **Keyboard navigation**: Arrow keys, Tab, Escape, Enter
- **ARIA attributes**: Roles, labels, and relationships
- **Screen reader announcements**: Live regions for dynamic content

### Labeling Requirements

Form controls must have accessible names:

```tsx
// Option 1: Wrap in label
<label>
  Email
  <Input />
</label>

// Option 2: Use Field component
<Field.Root>
  <Field.Label>Email</Field.Label>
  <Input />
  <Field.Error>Invalid email</Field.Error>
</Field.Root>

// Option 3: aria-label for icon buttons
<Dialog.Close aria-label="Close dialog">
  <CloseIcon />
</Dialog.Close>
```

### Keyboard Patterns

| Component | Key | Action |
|-----------|-----|--------|
| Dialog | `Escape` | Close dialog |
| Menu | `Arrow Up/Down` | Navigate items |
| Menu | `Enter/Space` | Select item |
| Tabs | `Arrow Left/Right` | Switch tabs |
| Select | `Arrow Up/Down` | Navigate options |
| Checkbox | `Space` | Toggle checked |

---

## 6. Common Patterns

### Opening Dialog from Menu

```tsx
function MenuWithDialog() {
  const [dialogOpen, setDialogOpen] = useState(false);

  return (
    <>
      <Menu.Root>
        <Menu.Trigger>Options</Menu.Trigger>
        <Menu.Portal>
          <Menu.Positioner>
            <Menu.Popup>
              <Menu.Item onClick={() => setDialogOpen(true)}>
                Delete...
              </Menu.Item>
            </Menu.Popup>
          </Menu.Positioner>
        </Menu.Portal>
      </Menu.Root>

      <Dialog.Root open={dialogOpen} onOpenChange={setDialogOpen}>
        <Dialog.Portal>
          <Dialog.Backdrop />
          <Dialog.Popup>
            <Dialog.Title>Confirm Delete</Dialog.Title>
            <Dialog.Close>Cancel</Dialog.Close>
            <button onClick={handleDelete}>Delete</button>
          </Dialog.Popup>
        </Dialog.Portal>
      </Dialog.Root>
    </>
  );
}
```

### Form with Validation

```tsx
function LoginForm() {
  return (
    <Form.Root onSubmit={handleSubmit}>
      <Field.Root>
        <Field.Label>Email</Field.Label>
        <Input name="email" type="email" required />
        <Field.Error match="valueMissing">
          Email is required
        </Field.Error>
        <Field.Error match="typeMismatch">
          Enter a valid email
        </Field.Error>
      </Field.Root>

      <Field.Root>
        <Field.Label>Password</Field.Label>
        <Input name="password" type="password" required />
        <Field.Error match="valueMissing">
          Password is required
        </Field.Error>
      </Field.Root>

      <button type="submit">Sign In</button>
    </Form.Root>
  );
}
```

### Hover-Triggered Popover

```tsx
<Popover.Root>
  <Popover.Trigger openOnHover delay={300}>
    Hover me
  </Popover.Trigger>
  <Popover.Portal>
    <Popover.Positioner>
      <Popover.Popup>
        Popover content
      </Popover.Popup>
    </Popover.Positioner>
  </Popover.Portal>
</Popover.Root>
```

### Nested Menus (Submenus)

```tsx
<Menu.Root>
  <Menu.Trigger>File</Menu.Trigger>
  <Menu.Portal>
    <Menu.Positioner>
      <Menu.Popup>
        <Menu.Item>New</Menu.Item>
        <Menu.Item>Open</Menu.Item>

        <Menu.SubmenuRoot>
          <Menu.SubmenuTrigger>Recent Files</Menu.SubmenuTrigger>
          <Menu.Portal>
            <Menu.Positioner>
              <Menu.Popup>
                <Menu.Item>file1.txt</Menu.Item>
                <Menu.Item>file2.txt</Menu.Item>
              </Menu.Popup>
            </Menu.Positioner>
          </Menu.Portal>
        </Menu.SubmenuRoot>
      </Menu.Popup>
    </Menu.Positioner>
  </Menu.Portal>
</Menu.Root>
```

### Controlled Tabs

```tsx
const [activeTab, setActiveTab] = useState('tab1');

<Tabs.Root value={activeTab} onValueChange={setActiveTab}>
  <Tabs.List>
    <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
    <Tabs.Tab value="tab2">Tab 2</Tabs.Tab>
    <Tabs.Indicator />
  </Tabs.List>
  <Tabs.Panel value="tab1">Content 1</Tabs.Panel>
  <Tabs.Panel value="tab2">Content 2</Tabs.Panel>
</Tabs.Root>
```

---

## 7. Component Reference

Full documentation for each component is available in `components/`:

### Overlays & Modals
- `components/dialog.md` - Modal windows
- `components/alert-dialog.md` - Confirmation modals
- `components/popover.md` - Floating content
- `components/tooltip.md` - Contextual hints
- `components/toast.md` - Notifications

### Navigation & Menus
- `components/menu.md` - Dropdown menus
- `components/menubar.md` - Horizontal menu bar
- `components/context-menu.md` - Right-click menus
- `components/navigation-menu.md` - Site navigation
- `components/tabs.md` - Tabbed content

### Form Controls
- `components/button.md` - Buttons
- `components/checkbox.md` - Checkboxes
- `components/checkbox-group.md` - Checkbox groups
- `components/radio.md` - Radio buttons
- `components/switch.md` - Toggle switches
- `components/toggle.md` - Toggle buttons
- `components/toggle-group.md` - Toggle button groups
- `components/input.md` - Text inputs
- `components/number-field.md` - Numeric inputs
- `components/select.md` - Dropdowns
- `components/combobox.md` - Searchable dropdowns
- `components/autocomplete.md` - Autocomplete inputs
- `components/slider.md` - Range sliders
- `components/field.md` - Form field wrapper
- `components/fieldset.md` - Field groups
- `components/form.md` - Form container

### Layout & Display
- `components/accordion.md` - Expandable sections
- `components/collapsible.md` - Single collapsible
- `components/scroll-area.md` - Custom scrollbars
- `components/separator.md` - Visual dividers
- `components/toolbar.md` - Action toolbars
- `components/avatar.md` - User avatars
- `components/meter.md` - Visual gauges
- `components/progress.md` - Progress indicators
- `components/preview-card.md` - Hover previews

---

## References

1. [Base UI Documentation](https://base-ui.com/)
2. [GitHub Repository](https://github.com/mui/base-ui)
3. [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
