# ⚡ Error Handling Best Practices

## 📋 Table of Contents

1. [Error Handling Best Practices — Server Side (Apex)](#1-error-handling-best-practices--server-side-apex)
2. [Error Handling Best Practices — Client Side (LWC JavaScript)](#2-error-handling-best-practices--client-side-lwc-javascript)
3. [Error Propagation & the Boundary Pattern](#3-error-propagation--the-boundary-pattern)
4. [Displaying & Logging Errors](#4-displaying--logging-errors)
5. [Interview Q&A (5-Year Level)](#5-interview-qa-5-year-level)

## 1. Error Handling Best Practices — Server Side (Apex)

### Why handle errors on the server side?
- Control **what** errors are sent to the client
- Control **how much detail** is exposed (abstraction/security)
- Handle certain errors entirely within Apex (e.g., callout exceptions)

### Three Approaches

#### 1. Unhandled (Bad Practice ❌)
```apex
@AuraEnabled
public static String toUpperCase(String str) {
    return str.toUpperCase(); // NullPointerException if str is null — exposed to client
}
```
Exposes full stack trace, class names, method names to the front-end.

#### 2. Aura Handled Exception (Better ✅)
```apex
@AuraEnabled
public static String toUpperCase(String str) {
    try {
        return str.toUpperCase();
    } catch (Exception e) {
        throw new AuraHandledException('Something went wrong: ' + e.getMessage());
    }
}
```
Returns only a clean message — no stack trace or internal details.

#### 3. Custom Exception Class (Best ✅✅)
```apex
public class MyException extends Exception {}

@AuraEnabled
public static String toUpperCase(String str) {
    try {
        return str.toUpperCase();
    } catch (Exception e) {
        throw new MyException('Custom error: ' + e.getMessage());
    }
}
```
Returns custom exception type + message + stack trace at the throw point.

#### 4. Anticipate and Avoid (Cleanest ✅✅✅)
```apex
@AuraEnabled
public static String toUpperCase(String str) {
    if (str != null) {
        return str.toUpperCase();
    }
    return ''; // No exception thrown at all
}

// Winter '21+: Safe Navigation Operator
return str?.toUpperCase(); // Returns null if str is null
```

### Passing Complex Error Data via Wrapper Class

```apex
public class ErrorWrapper {
    public String message;
    public String severity;
    public String correctiveAction;
}

// In catch block:
ErrorWrapper ew = new ErrorWrapper();
ew.message = 'Record not found';
ew.severity = 'HIGH';
ew.correctiveAction = 'Contact your admin';
throw new AuraHandledException(JSON.serialize(ew));
```

### Best Practices Summary — Apex

| Practice | Description |
|---|---|
| Use specific exception types | Catch `QueryException`, `DmlException` separately |
| Throw custom exceptions | Don't expose internal details to front-end |
| Standardize wrapper class | All developers use the same error structure |
| Decide per-error | Some errors should stay in Apex, not reach LWC |
| Safe Navigation Operator `?.` | Winter '21+, replaces null checks |

---

## 2. Error Handling Best Practices — Client Side (LWC JavaScript)

### 8.1 Synchronous Errors — `try-catch`

```javascript
connectedCallback() {
    try {
        console.log('first');
        throw new Error('Sync error');
        console.log('second'); // never reached
    } catch(e) {
        console.error('Caught:', e.message);
    }
    console.log('third'); // ✅ still runs
}
```

### 8.2 Asynchronous Errors — `try-catch` DOES NOT WORK across async boundaries

```javascript
// ❌ WRONG — catch will NOT catch errors inside setTimeout callback
connectedCallback() {
    try {
        setTimeout(() => {
            throw new Error('Async error'); // NOT caught by outer try-catch
        }, 1000);
    } catch(e) {
        // This never fires for the error inside setTimeout
    }
}

// ✅ CORRECT — put try-catch INSIDE the async callback
connectedCallback() {
    setTimeout(() => {
        try {
            throw new Error('Async error');
        } catch(e) {
            console.error('Caught inside callback:', e.message);
        }
    }, 1000);
}
```

> 🔑 **Key Rule:** `try-catch` only handles exceptions within the **same synchronous transaction**. Async callbacks run in a separate transaction.

### 8.3 Wire Service Errors

```javascript
import { wire } from 'lwc';
import getContacts from '@salesforce/apex/ContactController.getContacts';

export default class MyComponent extends LightningElement {
    // ✅ Always handle both data AND error
    @wire(getContacts)
    wiredContacts({ data, error }) {
        if (data) {
            this.contacts = data;
            // Handle data — wrap logic in try-catch for processing errors
            try {
                this.processData(data);
            } catch(e) {
                console.error('Processing error:', e);
            }
        } else if (error) {
            // ✅ ALWAYS include this else-if block
            this.error = error;
            console.error('Wire error:', error);
        }
    }
}
```

**For wire to property:**
```html
<!-- ✅ Check for error in HTML template -->
<template lwc:if={contacts.error}>
    <c-error-panel errors={contacts.error}></c-error-panel>
</template>
```

### 8.4 Imperative Apex Calls (Promises)

```javascript
import getContactList from '@salesforce/apex/ContactController.getContacts';

handleLoad() {
    getContactList({ accountId: this.recordId })
        .then(result => {
            this.contacts = result;
            // No need for try-catch here — errors propagate to .catch()
        })
        .catch(error => {
            // ✅ Catches errors from BOTH the Apex call AND the .then() block
            this.error = error;
        });
}
```

#### Promise Chaining — One catch block handles all

```javascript
getContext({ id: this.recordId })
    .then(context => getCases({ contextId: context.id }))
    .then(cases => getFiles({ caseIds: cases.map(c => c.Id) }))
    .catch(error => {
        // ✅ Catches errors from ALL steps above
        this.error = error;
        console.error('Chain error:', error);
    });
```

> ✅ **Best Practice:** Every promise chain must end with `.catch()`.

### 8.5 Errors in Component Lifecycle Hooks

```javascript
connectedCallback() {
    try {
        this.initializeSomething();
    } catch(e) {
        console.error('Init failed:', e);
    }
}
```

### 8.6 Avoid Inline Property Computation

```javascript
// ❌ WRONG — can't wrap in try-catch, unhandled if name is null
name = 'Raj';
fullName = this.name.toUpperCase(); // Error if name is null

// ✅ CORRECT — use a getter
get fullName() {
    return this.name ? this.name.toUpperCase() : '';
}
```

### 8.7 try-catch Scope — Granularity Matters

```javascript
// ❌ BAD — one giant try-catch hides which block failed
try {
    this.loadUser();
    this.loadCases();
    this.renderChart();
} catch(e) { console.log(e); }

// ✅ GOOD — separate blocks, each handled appropriately
try {
    this.loadUser();
} catch(e) {
    this.showUserError(e);
}

try {
    this.loadCases();
} catch(e) {
    this.showCasesError(e);
}
```

> 🔑 **Rule:** Handle only errors you **expect** in each catch block. **Propagate** unexpected errors upward.

---

## 3. Error Propagation & the Boundary Pattern

### Why propagate errors upward?

Lower-level (child) components often don't know **how** an error should be shown to the user. Higher-level (parent) components decide the UX.

### Method 1 — Throw keyword (synchronous propagation)

```javascript
// service/util JS (child)
export function divideNumbers(a, b) {
    if (b === 0) {
        throw new Error('Division by zero is not allowed');
    }
    return a / b;
}

// Parent component
import { divideNumbers } from 'c/mathUtils';

handleDivide() {
    try {
        const result = divideNumbers(this.numA, this.numB);
        this.result = result;
    } catch(e) {
        // Parent handles and decides how to show error
        this.dispatchEvent(new ShowToastEvent({
            title: 'Error',
            message: e.message,
            variant: 'error'
        }));
    }
}
```

### Method 2 — Custom Events (for child LWC components)

```javascript
// Child component
handleError() {
    if (this.numB === 0) {
        this.dispatchEvent(new CustomEvent('divisionerror', {
            detail: { message: 'Cannot divide by zero' }
        }));
    }
}
```

```html
<!-- Parent component template -->
<c-divide-component ondivisionerror={handleDivisionError}></c-divide-component>
```

```javascript
// Parent component JS
handleDivisionError(event) {
    this.errorMessage = event.detail.message;
    // Show toast, update UI, log to server, etc.
}
```

### The Boundary Component Pattern (Safeguard Pattern)

Wrap any component you want to protect inside a boundary component that implements `errorCallback()`:

```
App
└── c-boundary-component         ← implements errorCallback()
    └── c-risky-child-component  ← any unhandled error is caught above
```

**Benefits:**
- Catches all unexpected/unhandled errors from descendants
- Prevents broken UI (shows error panel instead)
- Single location to log errors to server
- Consistent error UI across the app

> 📌 You can wrap the **entire app** or just **individual sections** — architecture depends on risk areas.

---

## 4. Displaying & Logging Errors

### Error Structure Varies — Use `reduceErrors()`

Different error sources return different structures:

| Source | Structure |
|---|---|
| JavaScript runtime error | `{ name, message, stack }` |
| Apex unhandled exception | `{ body: { exceptionType, message, stackTrace } }` |
| Apex AuraHandledException | `{ body: { message } }` |
| LDS (createRecord etc.) | `{ body: { errors: [...] } }` |
| Custom Apex exception | `{ body: { exceptionType, message } }` |

**Solution:** Use `reduceErrors()` from [LWC Recipes](https://github.com/trailheadapps/lwc-recipes):

```javascript
import { reduceErrors } from 'c/ldsUtils';

// Normalizes ALL error types into a string[]
this.errors = reduceErrors(error);
```

### Displaying Errors — Best Practices

1. **Show errors near the point of failure** (inline validation messages)
2. **Always include corrective action** — don't just say "Error occurred", say what to do
3. **Use Base Lightning Components** — they are accessible by default (not just red color)

```javascript
// Set custom validity on a lightning-input
const el = this.template.querySelector('lightning-input');
el.setCustomValidity('Please enter a valid email address');
el.reportValidity();
```

4. **Use `c-error-panel` from LWC Recipes** for consistent UI:

```html
<c-error-panel errors={error}></c-error-panel>
```

5. **Be consistent** — same error display mechanism across the entire app

### Logging Errors

| Environment | Approach |
|---|---|
| Development | `console.error(error)` — preserves full stack trace |
| ❌ Never in production | `console.log` — no stack trace, security risk |
| Production | **Log to server** via Apex (custom Log object or Platform Events) |

```javascript
// ✅ Development logging
console.error('Error in handleLoad:', error);

// ✅ Production logging (log to Apex)
import logError from '@salesforce/apex/LogController.logError';

logError({ 
    message: error.message, 
    stack: error.stack, 
    component: 'myComponent' 
});
```

> ⚠️ **Security Warning:** Never leave `console.log` in production — it can expose sensitive data (tokens, OTPs, PII) to anyone who opens DevTools.

### Improper Handling Ranking (Worst → Best)

| Rank | Approach | Why |
|---|---|---|
| ❌ Worst | `try-catch` with only `console.log` | User sees nothing, error silently swallowed |
| ⚠️ Better | No handling at all | Lightning Runtime at least shows a popup |
| ✅ Best | `try-catch` with user feedback + server logging | User informed, error tracked |

---

## 5. Interview Q&A (5-Year Level)

**Q1: What is the order of lifecycle hook execution for a parent-child LWC pair?**

> `constructor (parent)` → `constructor (child)` → `connectedCallback (parent)` → `connectedCallback (child)` → `renderedCallback (child)` → `renderedCallback (parent)`

---

**Q2: Why can't you access child elements in `connectedCallback()`?**

> Because `connectedCallback()` fires when the component itself is inserted into the DOM, but child components haven't been rendered yet. Use `renderedCallback()` to access child DOM elements.

---

**Q3: Why does wrapping a `setTimeout` callback in an outer try-catch block not catch errors inside it?**

> `try-catch` only handles errors in the same synchronous execution context (transaction). `setTimeout` schedules the callback in a separate async transaction. The outer try-catch completes before the callback runs. The fix is to put the try-catch **inside** the callback.

---

**Q4: What is the `errorCallback()` hook, and what are its limitations?**

> `errorCallback(error, stack)` is a LWC-specific lifecycle hook that acts as an error boundary — it catches unhandled errors from **descendant** components in lifecycle hooks or template-declared event handlers. Limitations: it does NOT catch errors in the component itself, and does NOT catch errors from programmatically assigned event handlers (`addEventHandler` in JS).

---

**Q5: What happens to a child component when `errorCallback()` on the parent catches its error?**

> The child component is **unmounted and removed from the DOM**. The parent's `errorCallback()` then controls what to render instead (e.g., an error panel).

---

**Q6: What is the Safeguard/Boundary pattern in LWC?**

> Create a "boundary component" that wraps other components and implements `errorCallback()`. Any unhandled errors from wrapped descendants are caught here, logged, and an error UI is shown instead of a broken component. This enables consistent error handling across an app.

---

**Q7: Why should lower-level components throw errors instead of handling them?**

> Lower-level components (service components, deeply nested child components) don't know the appropriate UX response — whether to show a toast, an inline message, a modal, or log to server. Higher-level components have that context and should own the error-handling UX to ensure consistency.

---

**Q8: What does `.catch()` catch in a promise chain?**

> A single `.catch()` at the end of a promise chain catches errors from ALL `.then()` blocks in the chain — both the original async operation (e.g., Apex call) and any errors thrown inside the `.then()` handlers.

---

**Q9: What is `reduceErrors()` and why is it needed?**

> `reduceErrors()` is a utility function from LWC Recipes that normalizes errors from different sources (Apex, LDS, JavaScript, etc.) into a consistent `string[]`. This is needed because different error sources return different structures, making it hard to display or log them in a unified way.

---

**Q10: What is the difference between `render()` and `renderedCallback()` in LWC?**

> `render()` is a **protected method** on `LightningElement` that the framework calls during the render phase to determine **which HTML template** to display. It returns a template reference. `renderedCallback()` is a **lifecycle hook** that fires **after** the component has finished rendering — used for post-render DOM operations. You override `render()` to control *what* renders; you use `renderedCallback()` to react *after* it renders.

---

**Q11: When would you use `render()` instead of `lwc:if` for conditional rendering?**

> Use `render()` when you need to keep two entirely separate HTML templates in separate files — for example, when a component has two very different layouts with their own CSS, or when you want code-splitting for performance. For simple show/hide of elements within one template, `lwc:if|elseif|else` is simpler and preferred. Salesforce recommends `lwc:if` directives for most conditional rendering scenarios.

---

**Q12: What is the risk of leaving `console.log` in production LWC code?**

> It can expose sensitive data — like API tokens, session info, OTPs, or user PII — to anyone with DevTools access. It also adds no value for production monitoring. Production errors should always be logged to the server via Apex.

---

*SwiftNotes by Raj | Salesforce Platform Developer II Prep | April 2026*