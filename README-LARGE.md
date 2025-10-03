# CSS Apply Standard Proposal for W3C

## Abstract

This document proposes a new CSS feature called **CSS Block Variables (BlockVar)** and the **apply property** as a standard mechanism for declaring reusable style blocks and applying them to selectors. This proposal aims to improve CSS maintainability, transparency, and developer experience by providing a native, clear, and powerful pattern composition system.

## Table of Contents

1. [Introduction](#introduction)
2. [Motivation](#motivation)
3. [Syntax Overview](#syntax-overview)
4. [Core Concepts](#core-concepts)
5. [Use Cases and Scenarios](#use-cases-and-scenarios)
6. [Behavior Specifications](#behavior-specifications)
7. [Comparison with Existing Solutions](#comparison-with-existing-solutions)
8. [Benefits](#benefits)
9. [Examples](#examples)

---

## Introduction

CSS Block Variables (BlockVar) introduce a declarative way to define reusable blocks of CSS properties that can be applied to any selector using the `apply` property. This proposal extends CSS capabilities while maintaining backward compatibility and following CSS design principles of transparency and cascade.

### Key Features

- **Declarative style blocks** with `--blockname` syntax
- **Explicit application** with `apply: set(--blockname)`
- **Multiple inheritance** support
- **Pseudo-class variants** built-in
- **Context-aware** behavior within `@media`, `@supports`, etc.
- **Transparent resolution** - what you see is what you get

---

## Motivation

### Current Problems

1. **Repetition**: Developers repeat the same property sets across multiple selectors
2. **Maintenance burden**: Changing a common pattern requires multiple edits
3. **CSS Custom Properties limitations**: Variables work per-property, not per-block
4. **Utility frameworks complexity**: Tailwind and similar require learning new syntax
5. **Mixins absence**: CSS lacks native mixin support unlike preprocessors

### Why Not Existing Solutions?

| Solution | Limitation |
|----------|------------|
| CSS Variables | Only single values, not property blocks |
| @apply (PostCSS) | Non-standard, requires build step |
| Preprocessors | Build-time only, no runtime flexibility |
| Utility Classes | HTML pollution, learning curve |
| @extend (Sass) | Complex cascade issues, build-time only |

### Design Goals

1. **Transparency**: BlockVar definitions are clearly visible and understandable
2. **Simplicity**: Minimal new syntax, follows CSS patterns
3. **Flexibility**: Works with cascade, inheritance, and contexts
4. **Performance**: Can be optimized by browser engines
5. **Compatibility**: Degrades gracefully, polyfillable

---

## Syntax Overview

### Basic Syntax

```css
/* Define a BlockVar */
--button {
    padding: 12px 24px;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
}

/* Apply BlockVar to selector */
.primary-button {
    apply: set(--button);
    background: blue;
    color: white;
}
```

### With Pseudo-classes

```css
/* Define base state */
--button {
    padding: 12px 24px;
    background: #eee;
}

/* Define hover state */
--button:hover {
    background: #ddd;
    transform: translateY(-2px);
}

/* Apply automatically creates pseudo rules */
.my-button {
    apply: set(--button);
}
/* Automatically generates: .my-button:hover with --button:hover properties */
```

### Multiple Inheritance

```css
--base {
    margin: 0;
    padding: 0;
}

--rounded {
    apply: set(--base);
    border-radius: 8px;
}

--card {
    apply: set(--rounded);
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    background: white;
}

.product-card {
    apply: set(--card);
}
```

---

## Core Concepts

### 1. BlockVar Declaration

A BlockVar is declared with `--` prefix followed by a name and a block of CSS properties:

```css
--blockname {
    property: value;
    property: value;
}
```

**Rules:**
- BlockVar names must start with `--`
- Can contain letters, numbers, hyphens
- Case-sensitive
- Cannot be empty

### 2. The `apply` Property

The `apply` property references a BlockVar to apply its properties:

```css
selector {
    apply: set(--blockname);
}
```

**Behavior:**
- Properties from BlockVar are inserted at the position of `apply`
- Multiple `apply` can be used (first wins for conflicting properties)
- `apply` itself is removed from final computed styles
- Local properties override applied properties (cascade rules apply)

### 3. Inheritance Chain

BlockVars can reference other BlockVars:

```css
--parent {
    color: black;
}

--child {
    apply: set(--parent);
    font-size: 14px;
}
```

**Resolution order:**
1. Resolve parent BlockVars first (topological order)
2. Merge properties (child overrides parent)
3. Apply to target selector

### 4. Context Awareness

BlockVars respect CSS contexts:

```css
--button {
    padding: 12px;
}

@media (max-width: 768px) {
    --button {
        padding: 8px;
    }
}

.btn {
    apply: set(--button);
}
```

**Result:** `.btn` gets `padding: 12px` normally, `padding: 8px` in mobile context.

### 5. Pseudo-class Variants

BlockVars with pseudo-classes automatically generate matching rules:

```css
--link {
    color: blue;
}

--link:hover {
    color: darkblue;
}

a {
    apply: set(--link);
}
```

**Automatically generates:**
```css
a {
    color: blue;
}

a:hover {
    color: darkblue;
}
```

---

## Use Cases and Scenarios

### 1. Design System Tokens

```css
/* Define design tokens as BlockVars */
--spacing-small { padding: 8px; }
--spacing-medium { padding: 16px; }
--spacing-large { padding: 24px; }

--color-primary {
    background: #007bff;
    color: white;
}

--color-secondary {
    background: #6c757d;
    color: white;
}

/* Apply consistently across components */
.card-body {
    apply: set(--spacing-medium);
}

.primary-btn {
    apply: set(--color-primary);
    apply: set(--spacing-small);
}
```

### 2. Component Variants

```css
--button-base {
    display: inline-block;
    padding: 10px 20px;
    border: none;
    cursor: pointer;
    font-size: 16px;
    border-radius: 4px;
}

--button-primary {
    apply: set(--button-base);
    background: blue;
    color: white;
}

--button-secondary {
    apply: set(--button-base);
    background: gray;
    color: white;
}

--button-outline {
    apply: set(--button-base);
    background: transparent;
    border: 2px solid blue;
    color: blue;
}

.btn-primary { apply: set(--button-primary); }
.btn-secondary { apply: set(--button-secondary); }
.btn-outline { apply: set(--button-outline); }
```

### 3. Responsive Patterns

```css
--flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
}

--grid-responsive {
    display: grid;
    gap: 20px;
}

@media (min-width: 768px) {
    --grid-responsive {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (min-width: 1024px) {
    --grid-responsive {
        grid-template-columns: repeat(3, 1fr);
    }
}

.product-grid {
    apply: set(--grid-responsive);
}
```

### 4. State Management

```css
--interactive {
    cursor: pointer;
    transition: all 0.3s ease;
}

--interactive:hover {
    transform: scale(1.05);
}

--interactive:active {
    transform: scale(0.95);
}

--interactive:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    transform: none;
}

button {
    apply: set(--interactive);
}
```

### 5. Typography Systems

```css
--text-base {
    font-family: 'Inter', sans-serif;
    line-height: 1.5;
}

--heading {
    apply: set(--text-base);
    font-weight: 700;
    margin-bottom: 1rem;
}

--heading-xl {
    apply: set(--heading);
    font-size: 3rem;
}

--heading-lg {
    apply: set(--heading);
    font-size: 2rem;
}

--heading-md {
    apply: set(--heading);
    font-size: 1.5rem;
}

h1 { apply: set(--heading-xl); }
h2 { apply: set(--heading-lg); }
h3 { apply: set(--heading-md); }
```

### 6. Animation Patterns

```css
--fade-in {
    animation: fadeIn 0.3s ease-in;
}

--slide-up {
    animation: slideUp 0.4s ease-out;
}

--bounce {
    animation: bounce 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

.modal {
    apply: set(--fade-in);
}

.notification {
    apply: set(--slide-up);
}

.button {
    apply: set(--bounce);
}
```

---

## Behavior Specifications

### Normal Behavior

**Input:**
```css
--card {
    padding: 20px;
    background: white;
    border-radius: 8px;
}

.product {
    apply: set(--card);
    margin: 10px;
}
```

**Computed Result:**
```css
.product {
    padding: 20px;
    background: white;
    border-radius: 8px;
    margin: 10px;
}
```

**Explanation:**
1. BlockVar `--card` defines three properties
2. `apply: set(--card)` inserts those properties
3. Local property `margin: 10px` is preserved
4. Final selector has all properties combined

---

### Multiple Inheritance Behavior

**Input:**
```css
--reset {
    margin: 0;
    padding: 0;
}

--box {
    apply: set(--reset);
    display: block;
    box-sizing: border-box;
}

--card {
    apply: set(--box);
    background: white;
    padding: 20px; /* Override inherited padding: 0 */
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.article-card {
    apply: set(--card);
    max-width: 600px;
}
```

**Resolution Process:**

**Step 1:** Resolve `--reset`
```
margin: 0
padding: 0
```

**Step 2:** Resolve `--box` (inherits from `--reset`)
```
margin: 0          (from --reset)
padding: 0         (from --reset)
display: block     (own property)
box-sizing: border-box  (own property)
```

**Step 3:** Resolve `--card` (inherits from `--box`)
```
margin: 0          (from --reset via --box)
padding: 20px      (overrides --reset)
display: block     (from --box)
box-sizing: border-box  (from --box)
background: white  (own property)
box-shadow: 0 2px 4px rgba(0,0,0,0.1)  (own property)
```

**Step 4:** Apply to `.article-card`
```css
.article-card {
    margin: 0;
    padding: 20px;
    display: block;
    box-sizing: border-box;
    background: white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    max-width: 600px;
}
```

**Inheritance Rules:**
1. Child properties always override parent properties
2. Resolution happens depth-first (parents resolved before children)
3. Circular dependencies are invalid and must throw error
4. Multiple `apply` in same BlockVar are merged (first declaration wins)

---

### Pseudo-class Behavior

**Input:**
```css
--button {
    padding: 10px 20px;
    background: blue;
    color: white;
    border: none;
    cursor: pointer;
}

--button:hover {
    background: darkblue;
    transform: translateY(-2px);
}

--button:active {
    transform: translateY(0);
}

.submit-btn {
    apply: set(--button);
    font-size: 18px;
}
```

**Generated Output:**
```css
.submit-btn {
    padding: 10px 20px;
    background: blue;
    color: white;
    border: none;
    cursor: pointer;
    font-size: 18px;
}

.submit-btn:hover {
    background: darkblue;
    transform: translateY(-2px);
}

.submit-btn:active {
    transform: translateY(0);
}
```

**Pseudo-class Rules:**
1. When `apply: set(--blockname)` is used and `--blockname:pseudo` exists, automatically generate matching pseudo rules
2. Pseudo variants inherit all base properties first, then apply pseudo-specific overrides
3. Works with `:hover`, `:focus`, `:active`, `:visited`, `:disabled`, and other pseudo-classes
4. Multiple pseudo variants can coexist

**Complex Example with Inheritance:**

```css
--interactive {
    transition: all 0.3s;
}

--interactive:hover {
    opacity: 0.8;
}

--button {
    apply: set(--interactive);
    padding: 10px;
    background: blue;
}

--button:hover {
    background: darkblue;
}

.my-btn {
    apply: set(--button);
}
```

**Result:**
```css
.my-btn {
    transition: all 0.3s;    /* from --interactive */
    padding: 10px;           /* from --button */
    background: blue;        /* from --button */
}

.my-btn:hover {
    transition: all 0.3s;    /* inherited from --interactive */
    padding: 10px;           /* inherited from --button */
    background: darkblue;    /* from --button:hover, overrides base */
    opacity: 0.8;            /* from --interactive:hover */
}
```

**Note:** Pseudo-class variants inherit ALL properties from the inheritance chain, not just the base state.

---

## Comparison with Existing Solutions

### vs CSS Custom Properties

| Feature | CSS Variables | CSS Apply (BlockVar) |
|---------|--------------|---------------------|
| Scope | Single value | Block of properties |
| Syntax | `var(--name)` | `apply: set(--name)` |
| Inheritance | Via cascade | Explicit with `apply` |
| Grouping | No | Yes |
| Pseudo variants | Manual | Automatic |
| Runtime update | Yes | Yes (browser implementation) |

### vs @apply (PostCSS)

| Feature | PostCSS @apply | CSS Apply (Standard) |
|---------|---------------|---------------------|
| Browser support | Requires build | Native |
| Standardization | Plugin only | W3C proposal |
| Context awareness | Limited | Full |
| Pseudo automation | No | Yes |
| Runtime flexibility | No (compile-time) | Yes |

### vs Preprocessor Mixins

| Feature | Sass/Less Mixins | CSS Apply (BlockVar) |
|---------|-----------------|---------------------|
| Execution | Build-time | Browser runtime |
| Dynamic updates | No | Yes |
| Parameters | Yes | No (use CSS vars) |
| Visibility in DevTools | Compiled away | Native, visible |
| Learning curve | New syntax | Minimal (CSS-like) |

---

## Benefits

### 1. **Transparency and Clarity**

BlockVars are **explicit and visible** in the CSS source:

```css
--card {
    padding: 20px;
    background: white;
}
```

Unlike utility classes or cryptic abbreviations, BlockVar names are descriptive and the properties are immediately readable. Developers can understand what `--card` does by reading its definition.

### 2. **Maintainability**

Change once, apply everywhere:

```css
/* Update design system spacing */
--spacing-base {
    padding: 16px; /* Changed from 12px */
}

/* All components using --spacing-base update automatically */
```

### 3. **Reusability**

Define patterns once, reuse across projects:

```css
/* patterns.css - shared across apps */
--flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
}
```

### 4. **No Build Step Required**

Unlike preprocessors or PostCSS, this is native CSS. No compilation, no build configuration.

### 5. **Performance**

Browsers can optimize BlockVar resolution at parse time, potentially better than JavaScript-based solutions.

### 6. **Developer Experience**

- **Autocomplete**: IDEs can suggest BlockVar names
- **Jump to definition**: Navigate from `apply` to BlockVar definition
- **Validation**: Detect undefined BlockVars at dev time
- **DevTools**: Inspect applied properties with source BlockVar reference

### 7. **Composability**

Build complex patterns from simple blocks:

```css
--reset { margin: 0; padding: 0; }
--rounded { border-radius: 8px; }
--shadow { box-shadow: 0 2px 8px rgba(0,0,0,0.1); }

--card {
    apply: set(--reset);
    apply: set(--rounded);
    apply: set(--shadow);
    background: white;
}
```

### 8. **Context Sensitivity**

Works naturally with media queries and feature queries:

```css
--button { padding: 12px; }

@media (max-width: 768px) {
    --button { padding: 8px; }
}
```

No need for separate mobile classes or breakpoint utilities.

---

## Examples

### Example 1: Simple Design System

```css
/* tokens.css */
--text-sm { font-size: 14px; line-height: 1.4; }
--text-md { font-size: 16px; line-height: 1.5; }
--text-lg { font-size: 18px; line-height: 1.6; }

--color-primary { background: #007bff; color: white; }
--color-danger { background: #dc3545; color: white; }
--color-success { background: #28a745; color: white; }

/* components.css */
.button {
    apply: set(--text-md);
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

.button-primary {
    apply: set(--color-primary);
}

.button-danger {
    apply: set(--color-danger);
}
```

### Example 2: Complex Component System

```css
/* Base patterns */
--reset {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

--clickable {
    cursor: pointer;
    user-select: none;
}

--clickable:hover {
    opacity: 0.9;
}

--clickable:active {
    opacity: 0.8;
}

/* Component base */
--card-base {
    apply: set(--reset);
    background: white;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

--card-interactive {
    apply: set(--card-base);
    apply: set(--clickable);
    transition: transform 0.2s;
}

--card-interactive:hover {
    transform: translateY(-4px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}

/* Usage */
.product-card {
    apply: set(--card-interactive);
    max-width: 300px;
}

.info-card {
    apply: set(--card-base);
    border-left: 4px solid blue;
}
```

### Example 3: Responsive Grid System

```css
--grid {
    display: grid;
    gap: 20px;
    grid-template-columns: 1fr;
}

@media (min-width: 640px) {
    --grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

@media (min-width: 1024px) {
    --grid {
        grid-template-columns: repeat(3, 1fr);
    }
}

@media (min-width: 1280px) {
    --grid {
        grid-template-columns: repeat(4, 1fr);
    }
}

/* Usage */
.products {
    apply: set(--grid);
}

.articles {
    apply: set(--grid);
    gap: 30px; /* Override */
}
```

### Example 4: Form Styling

```css
--input-base {
    padding: 8px 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
    transition: border-color 0.2s;
}

--input-base:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 0 3px rgba(0,123,255,0.1);
}

--input-base:disabled {
    background: #f5f5f5;
    cursor: not-allowed;
}

--input-error {
    apply: set(--input-base);
    border-color: #dc3545;
}

--input-error:focus {
    border-color: #dc3545;
    box-shadow: 0 0 0 3px rgba(220,53,69,0.1);
}

/* Usage */
input[type="text"],
input[type="email"],
textarea {
    apply: set(--input-base);
}

input.error {
    apply: set(--input-error);
}
```

---

## Specification Details

### Parsing Rules

1. BlockVar declarations MUST start with `--` prefix
2. BlockVar declarations MUST only contain property declarations, not nested selectors
3. BlockVar names are case-sensitive
4. BlockVar can contain `apply` declarations to reference other BlockVars
5. Circular dependencies MUST be detected and treated as invalid

### Application Rules

1. `apply: set(--blockname)` MUST resolve the BlockVar at cascade time
2. Properties from BlockVar are inserted at the position of `apply` declaration
3. Multiple `apply` declarations are processed in order (first wins for conflicts)
4. Local properties override applied properties (standard cascade)
5. Undefined BlockVar references MUST be ignored (like undefined CSS variables)

### Context Rules

1. BlockVars are scoped to their context (`@media`, `@supports`, etc.)
2. When applying, search for BlockVar in current context first, then outer contexts
3. Context-specific BlockVars override global BlockVars with same name

### Pseudo-class Rules

1. When `apply: set(--name)` is used, check for `--name:pseudo` variants
2. For each pseudo variant found, automatically generate matching selector pseudo rule
3. Pseudo variants inherit from base BlockVar first, then apply pseudo-specific properties
4. Inheritance chains include pseudo variants from parent BlockVars

### Cascade Interaction

BlockVar application respects standard CSS cascade:

1. **Specificity**: Applied properties have the specificity of the target selector
2. **Origin**: Applied properties inherit the origin (author, user, UA) of the stylesheet
3. **Order**: Later `apply` can override earlier `apply` in same selector
4. **!important**: Applied properties can use `!important` flag

---

## Future Considerations

### Potential Extensions

1. **Parameterized BlockVars**: Allow passing values like mixins
   ```css
   --spacing($size) {
       padding: $size;
       margin: $size;
   }

   .box {
       apply: set(--spacing(20px));
   }
   ```

2. **Programmatic Updates**: JavaScript API to modify BlockVars
   ```javascript
   CSS.blockVars.set('--button', {
       padding: '12px 24px',
       background: 'blue'
   });
   ```

3. **BlockVar Exports**: Share BlockVars across shadow DOM boundaries
   ```css
   @export --button;
   ```

4. **Conditional Application**: Apply based on conditions
   ```css
   .element {
       apply: set(--card) if (min-width: 768px);
   }
   ```

---

## Conclusion

CSS Block Variables with the `apply` property provide a transparent, powerful, and native solution to CSS pattern composition. By following CSS design principles and maintaining backward compatibility, this proposal offers significant improvements to developer experience and code maintainability without requiring build tools or external dependencies.

The combination of **explicit BlockVar definitions**, **clear application syntax**, **multiple inheritance**, and **automatic pseudo-class handling** makes this proposal a natural extension to CSS that addresses real-world pain points while remaining simple and understandable.

We believe this standard would benefit the web platform and request consideration from the W3C CSS Working Group.

---

## References

- CSS Cascading and Inheritance Level 4
- CSS Custom Properties for Cascading Variables
- CSS Conditional Rules Module Level 3
- Sass @extend directive (prior art)

---

**Document Version**: 1.0
**Date**: 2025-10-02
**Status**: Proposal Draft
