# Base Lightning Components

> Working with Salesforce Data in Lightning Web Components — LWC Series, March 2025

---

## What Is Lightning Data Service (LDS)?

LDS is a centralized caching framework built on the **UI API**. It acts as an intelligent intermediary between your LWC and the Salesforce server.

| Feature | Description |
|---|---|
| **Caching** | Caches results on the client — reduces server round trips |
| **Security** | Respects CRUD, FLS visibility, and sharing settings |
| **Metadata** | Returns field metadata alongside record data |
| **Invalidation** | Auto-invalidates cache on data or metadata changes |

> Because Base Lightning Components are built on LDS, your components automatically inherit caching, security, and optimized server calls — without writing any data-access code.

---

## The Three Base Lightning Components

| Component | Use Case |
|---|---|
| `lightning-record-form` | Create + Edit + View in one component |
| `lightning-record-edit-form` | Create or Edit only (custom layout) |
| `lightning-record-view-form` | Read-only display (custom layout) |

---

## 1. `lightning-record-form`

The most versatile option. Handles **create, edit, view, and read-only** modes via a single `mode` attribute. Supports layout types and multi-column layouts.
```html
<!-- recordFormExample.html -->
<template>
    <lightning-record-form
        record-id={recordId}
        object-api-name="Contact"
        layout-type="Full"
        mode="view"
        fields={fields}>
    </lightning-record-form>
</template>
```
```js
// recordFormExample.js
import { LightningElement, api } from 'lwc';
import NAME_FIELD from '@salesforce/schema/Contact.Name';
import EMAIL_FIELD from '@salesforce/schema/Contact.Email';
import PHONE_FIELD from '@salesforce/schema/Contact.Phone';

export default class RecordFormExample extends LightningElement {
    @api recordId;

    fields = [NAME_FIELD, EMAIL_FIELD, PHONE_FIELD];
}
```

> 💡 Change `mode="view"` to `mode="edit"` or `mode="readonly"` as needed.

---

## 2. `lightning-record-edit-form`

Always renders in **edit state**. Use for creating new records or updating existing ones. Combine with `lightning-input-field` to control which fields appear.
```html
<!-- recordEditFormExample.html -->
<template>
    <lightning-record-edit-form
        record-id={recordId}
        object-api-name="Contact"
        onsuccess={handleSuccess}>

        <lightning-messages></lightning-messages>

        <lightning-input-field field-name="Name"></lightning-input-field>
        <lightning-input-field field-name="Email"></lightning-input-field>
        <lightning-input-field field-name="Phone"></lightning-input-field>

        <lightning-button
            type="submit"
            label="Save"
            variant="brand">
        </lightning-button>
    </lightning-record-edit-form>
</template>
```
```js
// recordEditFormExample.js
import { LightningElement, api } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

export default class RecordEditFormExample extends LightningElement {
    @api recordId;

    handleSuccess(event) {
        this.dispatchEvent(
            new ShowToastEvent({
                title: 'Success',
                message: 'Record saved successfully!',
                variant: 'success'
            })
        );
    }
}
```

> 💡 Leave `record-id` empty to use this component for **creating** a new record.

---

## 3. `lightning-record-view-form`

Always renders in **read-only** state. Use `lightning-output-field` to control which fields are displayed.
```html
<!-- recordViewFormExample.html -->
<template>
    <lightning-record-view-form
        record-id={recordId}
        object-api-name="Contact">

        <div class="slds-grid">
            <div class="slds-col slds-size_1-of-2">
                <lightning-output-field field-name="Name"></lightning-output-field>
                <lightning-output-field field-name="Email"></lightning-output-field>
            </div>
            <div class="slds-col slds-size_1-of-2">
                <lightning-output-field field-name="Phone"></lightning-output-field>
                <lightning-output-field field-name="AccountId"></lightning-output-field>
            </div>
        </div>

    </lightning-record-view-form>
</template>
```
```js
// recordViewFormExample.js
import { LightningElement, api } from 'lwc';

export default class RecordViewFormExample extends LightningElement {
    @api recordId;
}
```

---

## Feature Comparison

| Feature | `lightning-record-form` | `lightning-record-view-form` | `lightning-record-edit-form` |
|---|---|---|---|
| Create Records | ✅ | ❌ | ✅ |
| Edit Records | ✅ | ❌ | ✅ |
| View Records | ✅ | ✅ | ❌ |
| Read-Only Mode | ✅ | ✅ | ❌ |
| Layout Types | ✅ | ❌ | ❌ |
| Multi-Column Layout | ✅ | ❌ | ❌ |
| Custom Layout | ❌ | ✅ | ✅ |
| Custom Rendering | ❌ | ✅ | ✅ |

### Quick Selection Guide

- **Create + Edit + View in one component?** → `lightning-record-form`
- **Custom read-only layout?** → `lightning-record-view-form`
- **Controlled edit/create with validation?** → `lightning-record-edit-form`

---

## When NOT to Use Base Lightning Components

Move to **LDS Wire Adapters** or **Apex** when you need:

- Multi-step wizards or complex pre-save logic
- Multiple related records in one operation
- Non-standard UI deviating from Salesforce layout engine
- Fine-grained control over HTTP requests

---

## Summary

> Easiest to implement. Least code to write. Automatic caching, security, and validation. If your use case fits — Base Lightning Components are almost always the right first choice.

| Approach | Complexity | Flexibility |
|---|---|---|
| Base Lightning Components | Low | Low |
| LDS Wire Adapters | Medium | Medium |
| Apex | High | High |

---

*Salesforce LWC Developer Series • March 2025*
