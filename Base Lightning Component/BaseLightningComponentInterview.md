# Base Lightning Components — Interview Prep Guide
---

## Table of Contents

1. [What Are Base Lightning Components?](#1-what-are-base-lightning-components)
2. [Why Use Base Components Instead of Plain HTML?](#2-why-use-base-components-instead-of-plain-html)
3. [Form/Input Components](#3-forminput-components)
   - 3.1 [`lightning-input`](#31-lightning-input)
   - 3.2 [`lightning-combobox`](#32-lightning-combobox)
   - 3.3 [`lightning-button` Family](#33-lightning-button-family)
4. [Record-Aware Components — The Big Three](#4-record-aware-components--the-big-three)
   - 4.1 [`lightning-record-form`](#41-lightning-record-form)
   - 4.2 [`lightning-record-edit-form` + `lightning-input-field`](#42-lightning-record-edit-form--lightning-input-field)
   - 4.3 [`lightning-record-view-form` + `lightning-output-field`](#43-lightning-record-view-form--lightning-output-field)
   - 4.4 [Comparison Table](#44-comparison-table)
5. [`lightning-datatable` — The Interview Favorite](#5-lightning-datatable--the-interview-favorite)
   - 5.1 [Basic Setup](#51-basic-setup)
   - 5.2 [Row Actions](#52-row-actions)
   - 5.3 [Inline Editing](#53-inline-editing)
6. [Layout & Display Components](#6-layout--display-components)
   - 6.1 [`lightning-card`](#61-lightning-card)
   - 6.2 [`lightning-icon`](#62-lightning-icon)
   - 6.3 [`lightning-spinner`](#63-lightning-spinner)
   - 6.4 [`lightning-modal`](#64-lightning-modal)
7. [Common Interview Q&A](#7-common-interview-qa)
8. [Common Mistakes](#8-common-mistakes)
9. [Quick Reference Table](#9-quick-reference-table)

---

## 1. What Are Base Lightning Components?

Base Lightning Components are Salesforce's **pre-built, out-of-the-box** LWC/Aura components — things like `lightning-button`, `lightning-input`, `lightning-datatable`. They live in the `lightning` namespace and ship with Salesforce, meaning you never write their internal markup yourself.

**Interview framing:** *"Base components are Salesforce's ready-made building blocks that already implement SLDS (Salesforce Lightning Design System) styling, accessibility, and common behaviors — so I don't have to hand-build a styled button or a data table from scratch."*

---

## 2. Why Use Base Components Instead of Plain HTML?

| Reason | Plain HTML | Base Component |
|---|---|---|
| Styling | Manual SLDS classes needed | Built-in, consistent with Salesforce's own UI |
| Accessibility | You implement ARIA yourself | Accessibility handled internally |
| Validation | Manual | Built-in (`reportValidity()`, `required`, `pattern`, etc.) |
| Salesforce data awareness | None | Some components (record-form family) connect directly to Lightning Data Service |
| Maintenance | You own bug fixes | Salesforce maintains and updates them |

**Interview one-liner:** *"Use plain HTML only when no base component fits the need — Salesforce explicitly recommends checking base components first before reaching for `<input>` or `<button>`."*

---

## 3. Form/Input Components

### 3.1 `lightning-input`

The general-purpose input — supports many `type` values (`text`, `number`, `checkbox`, `date`, `email`, `toggle`, `search`, etc.)

```html
<lightning-input
    type="text"
    label="Full Name"
    value={name}
    required
    onchange={handleNameChange}>
</lightning-input>
```

```javascript
handleNameChange(event) {
    this.name = event.target.value;
}
```

> **Interview gotcha:** `event.target.value` works for `text`/`number`/etc., but for `type="checkbox"` or `type="toggle"`, you must read `event.target.checked` instead — `value` won't reflect the boolean state.

### 3.2 `lightning-combobox`

The dropdown/picklist component.

```html
<lightning-combobox
    label="Industry"
    value={selectedIndustry}
    options={industryOptions}
    onchange={handleIndustryChange}>
</lightning-combobox>
```

```javascript
industryOptions = [
    { label: 'Technology', value: 'Technology' },
    { label: 'Banking', value: 'Banking' }
];

handleIndustryChange(event) {
    this.selectedIndustry = event.detail.value;
}
```

> **Interview gotcha:** notice this reads `event.detail.value`, not `event.target.value` like `lightning-input` — a very common trick question. Combobox fires a custom event with the selected value in `.detail`.

### 3.3 `lightning-button` Family

| Component | Use case |
|---|---|
| `lightning-button` | Standard button, `variant` controls style (`brand`, `neutral`, `destructive`, etc.) |
| `lightning-button-icon` | Icon-only button |
| `lightning-button-menu` | Dropdown/menu button |
| `lightning-button-group` | Groups multiple buttons visually together |

```html
<lightning-button variant="brand" label="Save" onclick={handleSave}></lightning-button>
<lightning-button-icon icon-name="utility:delete" onclick={handleDelete}></lightning-button-icon>
```

---

## 4. Record-Aware Components — The Big Three

These three are the components interviewers ask about most, since they connect to **Lightning Data Service** automatically — no Apex, no `@wire`, no manual `getRecord` calls.

### 4.1 `lightning-record-form`

The simplest option — auto-generates a full form (view or edit mode) for a record, using either field names or a layout.

```html
<lightning-record-form
    record-id={recordId}
    object-api-name="Account"
    fields={fields}
    mode="edit">
</lightning-record-form>
```

```javascript
fields = ['Name', 'Industry', 'Phone'];
```

**Interview framing:** *"`lightning-record-form` is the fastest way to get a working record form — Salesforce builds the entire layout for you. I reach for it first, and only drop to `record-edit-form`/`record-view-form` when I need custom layout control."*

### 4.2 `lightning-record-edit-form` + `lightning-input-field`

More control than `record-form` — you build the layout yourself, field by field, but Salesforce still handles data binding, validation, and saving.

```html
<lightning-record-edit-form
    record-id={recordId}
    object-api-name="Account"
    onsuccess={handleSuccess}>
    <lightning-messages></lightning-messages>
    <lightning-input-field field-name="Name"></lightning-input-field>
    <lightning-input-field field-name="Industry"></lightning-input-field>
    <lightning-button type="submit" label="Save"></lightning-button>
</lightning-record-edit-form>
```

```javascript
handleSuccess(event) {
    console.log('Record saved, Id:', event.detail.id);
}
```

> **Interview gotcha:** `lightning-input-field` **only works inside** `lightning-record-edit-form` — it's not a standalone component. This trips up a lot of candidates.

### 4.3 `lightning-record-view-form` + `lightning-output-field`

Read-only sibling of the above — displays record data without allowing edits.

```html
<lightning-record-view-form record-id={recordId} object-api-name="Account">
    <lightning-output-field field-name="Name"></lightning-output-field>
    <lightning-output-field field-name="Industry"></lightning-output-field>
</lightning-record-view-form>
```

### 4.4 Comparison Table

| Component | Editable? | Layout control | Best for |
|---|---|---|---|
| `lightning-record-form` | Yes (via `mode`) | Low — auto-generated | Quick forms, minimal custom markup |
| `lightning-record-edit-form` | Yes | Full | Custom-arranged edit forms |
| `lightning-record-view-form` | No | Full | Custom-arranged read-only displays |

**Interview one-liner:** *"All three sit on top of Lightning Data Service — none of them need Apex or `@wire`. `record-form` trades control for speed; the other two trade speed for full layout control."*

---

## 5. `lightning-datatable` — The Interview Favorite

Almost every LWC interview touches this component — it's the standard way to render tabular Salesforce data with sorting, selection, and inline editing built in.

### 5.1 Basic Setup

```html
<lightning-datatable
    key-field="Id"
    data={accounts}
    columns={columns}
    onrowaction={handleRowAction}>
</lightning-datatable>
```

```javascript
columns = [
    { label: 'Name', fieldName: 'Name' },
    { label: 'Industry', fieldName: 'Industry' },
    { label: 'Phone', fieldName: 'Phone', type: 'phone' }
];
```

> **Interview gotcha:** `key-field` is **required** — it must match a unique field in your data (usually `Id`). Forgetting it either throws an error or breaks row identity tracking.

### 5.2 Row Actions

```javascript
columns = [
    { label: 'Name', fieldName: 'Name' },
    {
        type: 'action',
        typeAttributes: {
            rowActions: [
                { label: 'Edit', name: 'edit' },
                { label: 'Delete', name: 'delete' }
            ]
        }
    }
];

handleRowAction(event) {
    const actionName = event.detail.action.name;
    const row = event.detail.row;
    if (actionName === 'delete') {
        // delete logic
    }
}
```

### 5.3 Inline Editing

```javascript
columns = [
    { label: 'Name', fieldName: 'Name', editable: true }
];
```

```html
<lightning-datatable
    key-field="Id"
    data={accounts}
    columns={columns}
    draft-values={draftValues}
    onsave={handleSave}>
</lightning-datatable>
```

```javascript
handleSave(event) {
    const updatedFields = event.detail.draftValues;
    // pass updatedFields to updateRecord (from uiRecordApi) or Apex
}
```

**Interview one-liner:** *"Inline editing needs `editable: true` per column, plus `draft-values` bound in the template and an `onsave` handler — the datatable itself doesn't save anything, it just surfaces the edited values for you to persist."*

---

## 6. Layout & Display Components

### 6.1 `lightning-card`

The standard container — title, icon, header actions slot, footer.

```html
<lightning-card title="Account Details" icon-name="standard:account">
    <lightning-button slot="actions" label="Refresh"></lightning-button>
    <div class="slds-var-p-around_medium">Content here</div>
</lightning-card>
```

### 6.2 `lightning-icon`

Displays SLDS icons using the `category:icon-name` format.

```html
<lightning-icon icon-name="standard:account" size="small"></lightning-icon>
<lightning-icon icon-name="utility:delete" variant="error"></lightning-icon>
```

### 6.3 `lightning-spinner`

Loading indicator, usually paired with a boolean flag (from the Truthy/Falsy post earlier in this series).

```html
<lightning-spinner if:true={isLoading} alternative-text="Loading" size="medium"></lightning-spinner>
```

### 6.4 `lightning-modal`

The modern way to build popup dialogs (replacing older custom-built modal patterns).

```javascript
// myModal.js — extends LightningModal
import LightningModal from 'lightning/modal';

export default class MyModal extends LightningModal {
    handleClose() {
        this.close('closed');
    }
}
```

```javascript
// caller component
import MyModal from 'c/myModal';

async openModal() {
    const result = await MyModal.open({
        label: 'My Modal',
        size: 'small'
    });
    console.log(result);
}
```

---

## 7. Common Interview Q&A

**Q: What's the difference between `lightning-record-form` and building your own form with `getRecord`/`updateRecord`?**
> `lightning-record-form` needs zero custom JS for standard cases — Salesforce handles fetching, rendering, validating, and saving. Building your own with `getRecord`/`updateRecord` gives full control over UI but requires you to wire up every part manually. Use the base component first; drop to manual wiring only when custom logic/layout genuinely requires it.

**Q: How do you show a required-field error without a full form?**
> `lightning-input` has a `required` attribute and exposes `reportValidity()` — call it imperatively (e.g., on a button click) to trigger native validation UI without submitting a form.

```javascript
handleSave() {
    const inputField = this.template.querySelector('lightning-input');
    if (inputField.reportValidity()) {
        // proceed
    }
}
```

**Q: Can you style base Lightning components with custom CSS?**
> Only to a limited extent — base components render in their own shadow DOM boundary in some cases, so you can't reach inside with normal CSS selectors from the parent. Salesforce exposes specific **CSS custom properties (styling hooks)** for controlled customization instead of arbitrary overrides.

**Q: What does `lightning-record-edit-form`'s `onsuccess` actually give you?**
> The event's `.detail` contains the saved record's data, including `.id` — useful for navigating to the new record or updating local state after a successful save.

**Q: Why would you use `lightning-datatable` over a plain HTML table?**
> Built-in sorting, row selection, inline editing, infinite scrolling, and row actions — all without custom JS for the base behaviors. A plain HTML table needs every one of those hand-built.

---

## 8. Common Mistakes

| Mistake | Fix |
|---|---|
| Reading `event.target.value` on a `lightning-combobox` change | Combobox fires custom events — read `event.detail.value` instead |
| Reading `event.target.value` on a checkbox/toggle `lightning-input` | Use `event.target.checked` for boolean input types |
| Using `lightning-input-field` outside `lightning-record-edit-form` | It's not standalone — it only works nested inside that specific form component |
| Forgetting `key-field` on `lightning-datatable` | Required — must match a unique identifier field in the row data |
| Expecting `lightning-datatable` inline edits to save automatically | It only surfaces `draftValues` via the `onsave` event — you still call `updateRecord`/Apex yourself |
| Trying to deeply style base components with plain CSS selectors | Use documented CSS custom properties (styling hooks) instead — arbitrary selectors often can't pierce the component boundary |
| Choosing `lightning-record-edit-form` when `lightning-record-form` would do | Default to the simpler `record-form` unless custom layout is a genuine requirement |

---

## 9. Quick Reference Table

| Component | Category | Key Feature |
|---|---|---|
| `lightning-input` | Form | General-purpose input, many `type` values |
| `lightning-combobox` | Form | Dropdown; fires `event.detail.value` |
| `lightning-button` | Form | Standard action button, `variant` styling |
| `lightning-record-form` | Record-aware | Fastest way to a full record form |
| `lightning-record-edit-form` | Record-aware | Custom-layout editable form + `lightning-input-field` |
| `lightning-record-view-form` | Record-aware | Custom-layout read-only display + `lightning-output-field` |
| `lightning-datatable` | Data display | Sortable/selectable/editable table |
| `lightning-card` | Layout | Standard container with title/icon/actions slot |
| `lightning-icon` | Display | SLDS icon renderer |
| `lightning-spinner` | Display | Loading indicator |
| `lightning-modal` | Layout | Modern popup dialog pattern |

---
