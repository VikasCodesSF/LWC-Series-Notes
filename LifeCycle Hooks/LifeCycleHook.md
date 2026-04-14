# ⚡ LWC Lifecycle Hooks

## 📋 Table of Contents

1. [Lifecycle Hooks — Overview](#1-lifecycle-hooks--overview)
2. [constructor()](#2-constructor)
3. [connectedCallback() & disconnectedCallback()](#3-connectedcallback--disconnectedcallback)
4. [renderedCallback()](#4-renderedcallback)
5. [render()](#5-render)
6. [errorCallback()](#6-errorcallback)
7. [Lifecycle Flow — Quick Reference Table](#7-lifecycle-flow--quick-reference-table)
8. [Error Handling Best Practices — Server Side (Apex)](#8-error-handling-best-practices--server-side-apex)
9. [Error Handling Best Practices — Client Side (LWC JavaScript)](#9-error-handling-best-practices--client-side-lwc-javascript)
10. [Error Propagation & the Boundary Pattern](#10-error-propagation--the-boundary-pattern)
11. [Displaying & Logging Errors](#11-displaying--logging-errors)
12. [Interview Q&A (5-Year Level)](#12-interview-qa-5-year-level)

---

## 1. Lifecycle Hooks — Overview

A **lifecycle hook** is a callback method that fires at a specific phase of a component instance's lifecycle. LWC manages component creation, DOM insertion, rendering, and removal.

### Flow Direction Summary

| Hook | Flow Direction | Notes |
|---|---|---|
| `constructor()` | Parent → Child | Child elements don't exist yet |
| `render()` | Framework-invoked | Returns which HTML template to use |
| `connectedCallback()` | Parent → Child | Component inserted into DOM |
| `renderedCallback()` | **Child → Parent** | After every render/re-render |
| `disconnectedCallback()` | Parent → Child | Component removed from DOM |
| `errorCallback()` | Catches from **descendants** | Unique to LWC |

> 📌 `renderedCallback()` flows **child to parent** — opposite of the rest. `render()` is a protected method, not a true hook.

---

## 2. `constructor()`

### When it fires
When a component instance is **created** (before it's inserted into DOM).

### Flow
Parent → Child (parent fires first, children don't exist yet).

### Rules (from HTML Custom Elements spec)
- First statement **must** be `super()` — no parameters
- **Cannot** access child elements (they don't exist yet)
- **Cannot** access public properties (they're assigned after construction)
- **Cannot** add attributes to the host element here
- **Cannot** use `document.write()` or `document.open()`
- No `return` statement (except simple early-return)

### What to do here
- Initialize local/private state variables
- Call `super()` first, always

### Code Example

```javascript
import { LightningElement } from 'lwc';

export default class MyComponent extends LightningElement {
    isLoading;
    records = [];

    constructor() {
        super(); // MUST be first
        this.isLoading = true; // ✅ Initialize state
        // ❌ Don't do: this.template.querySelector(...) — too early
        // ❌ Don't do: this.setAttribute('data-id', '123') — use connectedCallback
    }
}
```

### ⚠️ Common Mistake
Adding attributes to the host element in constructor. Use `connectedCallback()` instead.

---

## 3. `connectedCallback()` & `disconnectedCallback()`

### `connectedCallback()`

#### When it fires
When the component is **inserted into the DOM**.

#### Flow
Parent → Child. Can fire **more than once** (e.g., if element is reordered in a list).

#### Use cases
- Fetch initial data from Apex
- Subscribe to Message Channel (LMS)
- Add event listeners to external elements (document/window)
- Initialize third-party libraries
- Set up caches
- Use `lightning/navigation`

> 💡 Properties are already assigned when `connectedCallback()` fires — unlike `constructor()`.

#### Code Example

```javascript
import { LightningElement } from 'lwc';
import { subscribe, MessageContext } from 'lightning/messageService';
import MY_CHANNEL from '@salesforce/messageChannel/MyChannel__c';

export default class MyComponent extends LightningElement {
    subscription = null;

    connectedCallback() {
        // ✅ Subscribe to message channel
        this.subscription = subscribe(
            this.messageContext,
            MY_CHANNEL,
            (message) => this.handleMessage(message)
        );

        // ✅ Fetch data
        this.loadData();
    }

    loadData() {
        // call Apex imperatively or wire
    }
}
```

#### ⚠️ If logic must run only once

```javascript
_initialized = false;

connectedCallback() {
    if (!this._initialized) {
        this._initialized = true;
        // one-time setup code
    }
}
```

---

### `disconnectedCallback()`

#### When it fires
When the component is **removed from the DOM**.

#### Flow
Parent → Child.

#### Use cases
- Unsubscribe from Message Channel
- Remove manually added event listeners (on `document`/`window`)
- Purge caches
- Clean up timers (`clearInterval`, `clearTimeout`)

#### Code Example

```javascript
disconnectedCallback() {
    // ✅ Unsubscribe from LMS
    unsubscribe(this.subscription);
    this.subscription = null;

    // ✅ Remove document-level listeners
    document.removeEventListener('keydown', this.handleKeydown);
}
```

> 🔑 `disconnectedCallback()` is the cleanup pair for `connectedCallback()`.

---

## 4. `renderedCallback()`

### When it fires
After **every render and re-render** of the component. Unique to LWC.

### Flow
**Child → Parent** (opposite of all other hooks).

### When a component re-renders
1. A reactive property/field changes
2. Component is marked "dirty"
3. A microtask queues the re-render

### Use cases
- One-time DOM manipulation after first render (e.g., initializing a chart library)
- Programmatically adding event listeners to template elements
- Interacting with third-party JS libraries that need the DOM to exist

### Code Example — One-time operation guard

```javascript
import { LightningElement } from 'lwc';
import chartjs from '@salesforce/resourceUrl/chartjs';
import { loadScript } from 'lightning/platformResourceLoader';

export default class MyChart extends LightningElement {
    hasRendered = false;

    renderedCallback() {
        if (this.hasRendered) return; // ✅ Guard against repeat execution
        this.hasRendered = true;

        loadScript(this, chartjs)
            .then(() => {
                this.initChart();
            })
            .catch(error => {
                console.error('Chart load failed', error);
            });
    }

    initChart() {
        const ctx = this.template.querySelector('canvas');
        // initialize chart...
    }
}
```

### ❌ What NOT to do in `renderedCallback()`

```javascript
renderedCallback() {
    // ❌ Updating a reactive property → causes INFINITE LOOP
    this.counter++; // triggers re-render → fires renderedCallback again → loop!

    // ❌ Updating a wire config object property
    this.wireParam = 'newValue'; // infinite loop risk
}
```

> ⚠️ **Golden Rule:** Never mutate reactive state inside `renderedCallback()` — it causes infinite render loops.

---

## 5. `render()`

### What it is
`render()` is a **protected method** on the `LightningElement` class — not technically a lifecycle hook, but it participates in the lifecycle. Unlike hooks (which may or may not exist on the prototype chain), `render()` **must** exist on the prototype chain.

### When to use it
Rarely needed. The primary use case is **conditionally rendering a different HTML template** based on component state — when you want to keep two very different UI layouts in separate HTML files rather than mixing them with `lwc:if` directives in one file.

> 💡 Salesforce recommends preferring `lwc:if|elseif|else` directives for conditional rendering. Use `render()` only when you genuinely need **separate HTML files** (e.g., code-splitting two very different layouts).

### When it is called
Can be called **before or after** `connectedCallback()` — the framework invokes it during the render phase to determine which template to display.

### Rules
- Must **return a valid HTML template reference** (the imported default export from an `.html` file)
- Each extra template HTML file can only use a CSS file with the **same filename** (e.g., `templateTwo.html` → `templateTwo.css`)
- The default `render()` returns the component's main HTML file (`componentName.html`) unless overridden

### Code Example — Multiple Templates

**Component bundle structure:**
```
myComponent/
├── myComponent.html        ← default template (not used when render() is overridden)
├── myComponent.js
├── templateOne.html        ← template for view mode
├── templateOne.css
├── templateTwo.html        ← template for edit mode
└── templateTwo.css
```

**templateOne.html**
```html
<template>
    <div class="view-mode">
        <p>{recordName}</p>
        <lightning-button label="Edit" onclick={handleEdit}></lightning-button>
    </div>
</template>
```

**templateTwo.html**
```html
<template>
    <div class="edit-mode">
        <lightning-input label="Name" value={recordName} onchange={handleChange}></lightning-input>
        <lightning-button label="Save" onclick={handleSave}></lightning-button>
    </div>
</template>
```

**myComponent.js**
```javascript
import { LightningElement } from 'lwc';
import templateOne from './templateOne.html';   // ✅ import both templates
import templateTwo from './templateTwo.html';

export default class MyComponent extends LightningElement {
    isEditMode = false;
    recordName = 'Raj';

    // ✅ render() returns the correct template based on state
    render() {
        return this.isEditMode ? templateTwo : templateOne;
    }

    handleEdit() {
        this.isEditMode = true;
    }

    handleSave() {
        this.isEditMode = false;
    }

    handleChange(event) {
        this.recordName = event.target.value;
    }
}
```

### `render()` vs `lwc:if` — When to use which?

| Scenario | Use |
|---|---|
| Simple show/hide of elements | `lwc:if` / `lwc:else` in template |
| Two entirely different layouts in one file | `lwc:if` blocks (still fine for most cases) |
| Two very different layouts — want separate HTML files, separate CSS, code splitting | `render()` method |
| More than 2 templates based on complex state | `render()` with multiple imported templates |

### ⚠️ Important Notes
- CSS scoping: `templateTwo.html` can only use `templateTwo.css` — it **cannot** inherit CSS from `myComponent.css` or `templateOne.css`
- Switching templates via `render()` causes the component to **re-render**, which fires `renderedCallback()` again
- Don't use `render()` just because you learned about it — `lwc:if` is simpler and covers 90% of cases

---

## 6. `errorCallback()`

### When it fires
When an **error is thrown by a descendant component** — in its lifecycle hooks or in an HTML-template-declared event handler.

### Flow
Catches errors from **child/descendant components** only (not the component itself).

### Unique to LWC
This is a LWC-specific lifecycle hook, not part of standard Web Components.

### Signature

```javascript
errorCallback(error, stack) {
    // error → JavaScript native Error object
    // stack → string with stack trace info
}
```

### What it catches ✅
- Errors in lifecycle hooks of descendant components
- Errors in event handlers declared in the HTML template (`onclick`, `onchange`, etc.)

### What it does NOT catch ❌
- Errors in event handlers assigned programmatically via `addEventHandler()` in JavaScript
- Errors occurring within the component itself

### ⚠️ Important Behavior
When `errorCallback()` catches an error, the **child component that threw the error is unmounted and removed from the DOM**.

### Code Example — Error Boundary Pattern

**boundaryComponent.html**
```html
<template>
    <template lwc:if={error}>
        <c-error-panel errors={error}></c-error-panel>
    </template>
    <template lwc:else>
        <slot></slot>
    </template>
</template>
```

**boundaryComponent.js**
```javascript
import { LightningElement } from 'lwc';

export default class BoundaryComponent extends LightningElement {
    error;

    errorCallback(error, stack) {
        this.error = error;
        console.error('Error caught by boundary:', error);
        console.error('Stack:', stack);
        // Optionally: log to server here
    }
}
```

**Usage — wrap any component you want to guard:**
```html
<c-boundary-component>
    <c-my-risky-component></c-my-risky-component>
</c-boundary-component>
```

---

## 7. Lifecycle Flow — Quick Reference Table

| Hook | Fires When | Direction | Access `this.template`? | Can access child elements? |
|---|---|---|---|---|
| `constructor()` | Component created | Parent → Child | ❌ No | ❌ No |
| `render()` | During render phase | — (framework-invoked) | ❌ No | ❌ No |
| `connectedCallback()` | Inserted into DOM | Parent → Child | ✅ Yes | ❌ No (children not yet rendered) |
| `renderedCallback()` | After render/re-render | **Child → Parent** | ✅ Yes | ✅ Yes |
| `disconnectedCallback()` | Removed from DOM | Parent → Child | ✅ Yes | ✅ Yes |
| `errorCallback(error, stack)` | Descendant throws | Catches from descendants | ✅ Yes | — |

> 📌 `render()` is a **protected method**, not a true hook — but it participates in the render lifecycle.

---