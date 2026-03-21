# ⚡ LWC – Working with Data (Base Lightning Components)

## 1. Introduction – Ways to Work with Salesforce Data in LWC

There are **3 main approaches** to interact with Salesforce data in LWC:

| Approach | Ease | Flexibility |
|---|---|---|
| **Base Lightning Components** (built on LDS) | Easiest to implement | Less flexible |
| **LDS Wire Adapters & Functions** | Easy to use | More flexible than Base Components |
| **Apex** | More complex | Extremely flexible |

> **Rule of Thumb:** Use the simplest approach that meets your requirements. Base components → LDS Wire → Apex (in order of preference).

---

## 2. Lightning Data Service (LDS)

### What is LDS?
- LDS is a **centralized data caching framework** built on top of the **User Interface API (UI API)**.
- It sits between the browser (LWC) and the Salesforce server.

### How LDS Works (Caching Flow)
```
LWC requests Record A
    → LDS checks cache
        → If NOT in cache → Goes to Server (UI API → Database) → Saves in cache → Returns data
        → If IN cache → Fetches from cache directly → Returns data
```

### Key Properties of LDS
1. Built on top of **UI API**
2. UI API gives data + metadata in a **single response**; respects CRUD access, FLS, and sharing settings
3. LDS displays only records/fields for which users have **CRUD access and FLS visibility**
4. **Caches results on the Client** (browser)
5. **Invalidates cache** when Salesforce data/metadata changes (use `notifyRecordUpdateAvailable()` to manually trigger this)
6. **Optimizes server calls** (avoids redundant network requests)

> **`notifyRecordUpdateAvailable()`** — Call this method to inform LDS that record data has changed externally (e.g., after an Apex DML), so LDS can refresh wire adapters with the latest data.

---

## 3. Base Lightning Components

Base Lightning Components are **built on top of LDS** and inherit its caching and synchronization capabilities.

### The 3 Base Components

| Component | Purpose |
|---|---|
| `lightning-record-form` | Create, view, and edit records. Least customizable. |
| `lightning-record-view-form` | Display (read-only) a record with custom layout |
| `lightning-record-edit-form` | Create or edit a record with custom layout |

### When to Use These Components?
- Build a metadata-driven or form-based UI similar to Salesforce record detail page
- Display record values based on field metadata
- Hide or show localized field labels
- Display help text on a custom field
- Perform client-side validation and enforce validation rules

### Feature Comparison Table

| Feature | `lightning-record-form` | `lightning-record-view-form` | `lightning-record-edit-form` |
|---|:---:|:---:|:---:|
| Create Records | ✅ | ❌ | ✅ |
| Edit Records | ✅ | ❌ | ✅ |
| View Records | ✅ | ✅ | ❌ |
| Read-Only Mode | ✅ | ✅ | ❌ |
| Layout Types | ✅ | ❌ | ❌ |
| Multi Column Layout | ✅ | ❌ | ❌ |
| Custom Layout for Fields | ❌ | ✅ | ✅ |
| Custom Rendering of Record Data | ❌ | ✅ | ✅ |

---

## 4. `lightning-record-form`

### Overview
- Used to **quickly create forms** to add, view, or update a record.
- Less customizable than `lightning-record-edit-form` or `lightning-record-view-form`.
- Automatically switches between view and edit modes.
- Provides Cancel and Save buttons automatically in edit forms.

### Key Attributes

| Attribute | Description |
|---|---|
| `object-api-name` | **Always required.** Establishes the relationship between record and object. Note: **Event and Task objects are NOT supported.** |
| `record-id` | Required only when **editing or viewing** an existing record. |
| `fields` | Array of field API name strings. Fields display in the order listed. |
| `layout-type` | `Full` or `Compact`. Defined by administrators. ⚠️ Use either `layout-type` OR `fields`, not both. |
| `mode` | `edit` (default when no record-id), `view` (default when record-id given), `readonly` |
| `columns` | Number of columns for multi-column layout |

### 3 Modes
- **edit** – Creates an editable form. Default when `record-id` is NOT provided (for new records).
- **view** – Creates a form to display a record with inline editing. Default when `record-id` IS provided.
- **readonly** – Displays a record that the user cannot edit.

### How to Import Fields
```javascript
// Import object
import ACCOUNT_OBJECT from '@salesforce/schema/Account';

// Import fields
import NAME_FIELD from '@salesforce/schema/Account.Name';
import ANNUAL_REVENUE_FIELD from '@salesforce/schema/Account.AnnualRevenue';
import TYPE_FIELD from '@salesforce/schema/Account.Type';
import INDUSTRY_FIELD from '@salesforce/schema/Account.Industry';
```

### Example: Create a Record
**HTML:**
```html
<template>
    <lightning-card title="Create record using lightning-record-form">
        <lightning-record-form
            object-api-name={objectName}
            fields={fieldList}
            onsuccess={handleSuccess}>
        </lightning-record-form>
    </lightning-card>
</template>
```

**JS:**
```javascript
import { LightningElement } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';
import NAME_FIELD from '@salesforce/schema/Account.Name';
import ANNUAL_REVENUE_FIELD from '@salesforce/schema/Account.AnnualRevenue';
import TYPE_FIELD from '@salesforce/schema/Account.Type';
import INDUSTRY_FIELD from '@salesforce/schema/Account.Industry';

export default class RecordFormDemo extends LightningElement {
    objectName = ACCOUNT_OBJECT;
    fieldList = [NAME_FIELD, ANNUAL_REVENUE_FIELD, TYPE_FIELD, INDUSTRY_FIELD];

    handleSuccess(event) {
        console.log(event.detail.id);
        const toastEvent = new ShowToastEvent({
            title: 'Account Created',
            message: 'Record id' + event.detail.id + 'was created',
            variant: 'success'
        });
        this.dispatchEvent(toastEvent);
    }
}
```

### Example: Edit a Record (from Record Page using @api)
```javascript
// When placed on a Record Page, @api recordId is auto-populated by Salesforce engine
@api recordId;
@api objectApiName;
```

```html
<lightning-record-form
    record-id={recordId}
    object-api-name={objectApiName}
    fields={fieldList}
    mode="edit"
    columns="2">
</lightning-record-form>
```

### Example: Display (View) a Record
```html
<!-- View mode (with inline edit icons) -->
<lightning-record-form
    record-id="0011o000002niCSzAAM"
    object-api-name={objectName}
    fields={fieldList}>
</lightning-record-form>

<!-- Readonly mode (no edit icons or buttons) -->
<lightning-record-form
    record-id="0011o000002niCSzAAM"
    object-api-name={objectName}
    fields={fieldList}
    mode="readonly">
</lightning-record-form>
```

### Example: Edit with Layout Type
```html
<lightning-record-form
    record-id="Your Record Id"
    object-api-name="Your Object API Name"
    columns="2"
    mode="edit"
    layout-type="Compact">
</lightning-record-form>
```
> ⚠️ **Important:** Use only **one** of either `layout-type` OR `fields` — not both at the same time.

### Q&A Notes
- **Q: Can we use Array instead of Object to map fields?**  
  A: Yes, you can use either. If you use Array, you need to use a loop to iterate.

- **Q: Is `@api recordId` auto-populated on a Record Page?**  
  A: Yes. When the component is placed on a Record Page, Salesforce engine auto-fetches the `recordId`.

- **Q: What if I have 200 fields — do I have to import all of them?**  
  A: Yes, you must import all 200 fields for LWC. Alternatively, use **Apex** for heavy field loads (preferred). You can use `getObjectInfo` to just fetch field names dynamically, but for heavy data, Apex is recommended.

- **Q: Is `lightning-record-form` the child or parent?**  
  A: `lightning-record-form` is the **child** component; the component embedding it (e.g., `recordFormDemo`) is the **parent**. When `onsuccess` fires, the child (`lightning-record-form`) passes data up to the parent via `event.detail`.

---

## 5. `lightning-record-view-form`

### Overview
- Use to **display Salesforce record data** for specified fields associated with a record.
- Fields are rendered with their labels and current values as **read-only**.
- Use `lightning-output-field` components inside to specify which fields to display.
- For customized layout, use this. For simple display without customization, use `lightning-record-form`.

### Key Attributes
| Attribute | Required | Description |
|---|---|---|
| `record-id` | Yes | The ID of the record to display |
| `object-api-name` | Yes (best practice) | The API name of the object |

> **Best practice:** Always import `object-api-name` just like `lightning-record-form` (even though it can work without it, it's recommended).

### Example
**HTML:**
```html
<template>
    <lightning-card title="lightning-record-view-form">
        <lightning-record-view-form
            record-id="0011y00000pch2VAAQ"
            object-api-name="Account">
            <div class="slds-grid slds-gutters">
                <div class="slds-col slds-size_6-of-12">
                    <lightning-output-field field-name="Name"></lightning-output-field>
                    <lightning-output-field field-name="Type"></lightning-output-field>
                    <lightning-output-field field-name="Industry"></lightning-output-field>
                </div>
                <div class="slds-col slds-size_6-of-12">
                    <lightning-output-field field-name="AnnualRevenue"></lightning-output-field>
                </div>
            </div>
        </lightning-record-view-form>
    </lightning-card>
</template>
```

### Q&A Notes
- **Q: Can we display multiple records with `lightning-record-view-form`?**  
  A: In real-time scenarios, these forms are used for **single records**. For multiple records, prefer **custom components** because error handling is easier.

- **Q: Can we display parent (lookup) fields, e.g., Account fields from Contact?**  
  A: This requires fetching the lookup field value. You can use the lookup field reference in the field name.

---

## 6. `lightning-record-edit-form`

### Overview
- Displays an **editable form** for creating or editing a Salesforce record.
- Provides **custom layout** of fields and custom rendering of record data.
- You can intercept the form submit using `onsubmit` to add custom validation before the record is saved.

### Key Attributes
| Attribute | Description |
|---|---|
| `object-api-name` | API name of the Salesforce object |
| `record-id` | Pass a record ID to **edit** an existing record. Omit for **creating** a new record. |
| `onsubmit` | Handler fired when the form is submitted |
| `onsuccess` | Handler fired on successful record save |
| `onerror` | Handler fired when an error occurs |

### Child Components Used Inside
- `lightning-messages` — Displays form-level error messages
- `lightning-input-field` — Editable input for a specific field
- `lightning-output-field` — Read-only display of a field
- `lightning-button` (type="submit") — Submits the form

### Example: Create a Record
**HTML:**
```html
<template>
    <lightning-card title="lightning-record-edit-form">
        <lightning-record-edit-form
            object-api-name={objectName}
            onsubmit={handleSubmit}
            onsuccess={successHandler}
            onerror={handleError}>
            <lightning-messages></lightning-messages>
            <lightning-input-field field-name={fields.accountField}></lightning-input-field>
            <lightning-input-field field-name={fields.nameField}></lightning-input-field>
            <lightning-input-field field-name={fields.titleField}></lightning-input-field>
            <lightning-input-field field-name={fields.phoneField}></lightning-input-field>
            <lightning-input-field field-name={fields.emailField}></lightning-input-field>
            <lightning-button class="slds-m-around-xx_small" label="Cancel"></lightning-button>
            <lightning-button variant="brand" type="submit"
                class="slds-m-around-xx_small" label="Save"></lightning-button>
        </lightning-record-edit-form>
    </lightning-card>
</template>
```

**JS:**
```javascript
import { LightningElement } from 'lwc';
import CONTACT_OBJECT from '@salesforce/schema/Contact';
import NAME_FIELD from '@salesforce/schema/Contact.Name';
import TITLE_FIELD from '@salesforce/schema/Contact.title';
import PHONE_FIELD from '@salesforce/schema/Contact.phone';
import EMAIL_FIELD from '@salesforce/schema/Contact.Email';
import ACCOUNT_FIELD from '@salesforce/schema/Contact.AccountId';

export default class RecordEditRecord extends LightningElement {
    objectName = CONTACT_OBJECT;
    fields = {
        accountField: ACCOUNT_FIELD,
        nameField: NAME_FIELD,
        titleField: TITLE_FIELD,
        phoneField: PHONE_FIELD,
        emailField: EMAIL_FIELD
    };
}
```

> **Note:** You can use an **Object** (as above) or an **Array** to hold field references. If using Array, iterate with a loop.

### Example: Edit an Existing Record
Pass the `record-id` attribute — the form automatically fetches all the data and maps it to the respective fields based on their names.

```html
<lightning-record-edit-form
    object-api-name={objectName}
    record-id="0031y00000coorMAAQ">
    <lightning-messages></lightning-messages>
    <lightning-input-field field-name={fields.nameField}></lightning-input-field>
    <!-- more fields -->
    <lightning-button variant="brand" type="submit"
        class="slds-m-around-xx_small" label="Save">
    </lightning-button>
</lightning-record-edit-form>
```

---

## 7. Reset the `lightning-record-edit-form`

To reset all input fields in the form, use `querySelectorAll` to get all `lightning-input-field` elements and call `.reset()` on each.

**JS:**
```javascript
handleReset() {
    const inputFields = this.template.querySelectorAll('lightning-input-field');
    if (inputFields) {
        Array.from(inputFields).forEach(field => {
            field.reset();
        });
    }
}
```

**HTML (add a Reset button):**
```html
<lightning-button onclick={handleReset} class="slds-m-around-xx_small" label="Cancel">
</lightning-button>
<lightning-button variant="brand" type="submit" class="slds-m-around-xx_small" label="Save">
</lightning-button>
```

---

## 8. Adding Custom Labels to Fields

By default, `lightning-input-field` uses the field's metadata label. To override the label, hide the default label using `variant="label-hidden"` and add a custom `<label>` element above it.

```html
<label class="slds-p-left_x-small">Enter the email</label>
<lightning-input-field variant="label-hidden" field-name={fields.emailField}>
</lightning-input-field>
```

This renders the custom label ("Enter the email") above the field instead of the default metadata label.

---

## 9. Custom Validation in `lightning-record-edit-form`

Use `onsubmit` to intercept the form submission and apply custom validation logic before saving.

### Steps:
1. Prevent the default form submission with `event.preventDefault()`
2. Get the input component using `querySelector`
3. Apply validation with `setCustomValidity()` — pass error message string if invalid, or empty string `""` if valid
4. If valid, manually submit with `.submit(fields)`
5. Call `reportValidity()` to display the validation state

### Full Example
**HTML (`recordidCustom.html`):**
```html
<template>
    <lightning-card title="Custom validation in lightning record edit form">
        <lightning-record-edit-form
            object-api-name={objectName}
            onsubmit={handleSubmit}
            onsuccess={successHandler}
            onerror={handleError}>
            <lightning-input
                label="Name"
                value={inputValue}
                onkeyup={handleChange}
                class="slds-m-bottom_x-small">
            </lightning-input>
            <lightning-button class="slds-m-top_small" type="submit" label="Create Account">
            </lightning-button>
        </lightning-record-edit-form>
    </lightning-card>
</template>
```

**JS (`recordidCustom.js`):**
```javascript
import { LightningElement } from 'lwc';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

export default class RecordEditCustom extends LightningElement {
    objectName = ACCOUNT_OBJECT;
    inputValue = '';

    handleChange(event) {
        this.inputValue = event.target.value;
    }

    handleSubmit(event) {
        event.preventDefault();
        const inputCmp = this.template.querySelector('lightning-input');
        const value = inputCmp.value;

        if (!value.includes('Australia')) {
            // Validation fails — set error message
            inputCmp.setCustomValidity("The account name must include 'Australia'");
        } else {
            // Validation passes — clear error and submit
            inputCmp.setCustomValidity("");
            const fields = event.detail.fields;
            fields.Name = value;
            this.template.querySelector('lightning-record-edit-form').submit(fields);
        }
        inputCmp.reportValidity(); // Show validation state
    }

    successHandler(event) {
        const toastEvent = new ShowToastEvent({
            title: "Account created",
            message: "Record ID: " + event.detail.id,
            variant: "success"
        });
        this.dispatchEvent(toastEvent);
    }

    handleError(event) {
        const toastEvent = new ShowToastEvent({
            title: "Error creating Account",
            message: event.detail.message,
            variant: "error"
        });
        this.dispatchEvent(toastEvent);
    }
}
```

### Key Methods for Validation
| Method | Description |
|---|---|
| `setCustomValidity("error message")` | Sets a custom error message on the field |
| `setCustomValidity("")` | Clears the custom error (marks field as valid) |
| `reportValidity()` | Displays the current validity state to the user |

---

## 10. Quick Summary – Which Component to Use?

| Scenario | Best Component |
|---|---|
| Quick create/edit form, admin-configurable fields | `lightning-record-form` |
| Display a single record (read-only) with custom layout | `lightning-record-view-form` |
| Create a new record with custom layout + validation | `lightning-record-edit-form` (no record-id) |
| Edit an existing record with custom layout + validation | `lightning-record-edit-form` (with record-id) |
| Complex multi-record display | Custom component + Apex |
| Handling large number of fields (100+) | Apex + custom component |

---

## 11. Important Tips & Gotchas

- `lightning-record-form` does **not** support **Event and Task** objects.
- **Do not use both** `layout-type` and `fields` in `lightning-record-form` — pick one.
- For `lightning-record-view-form`, import `object-api-name` as a best practice even though it's optional.
- When component is on a **Record Page**, `@api recordId` and `@api objectApiName` are auto-populated by the Salesforce engine.
- `lightning-record-form` is a **less customizable** wrapper. Use `lightning-record-edit-form` + `lightning-record-view-form` when you need custom field layout or rendering.
- For **multiple records**, avoid base record forms — use custom components with Apex for better error handling.
- To dynamically get all field names of an object (without importing individually), use the `getObjectInfo` wire adapter. But for heavy data loads, use **Apex**.

---

*📌 Reference: [Salesforce LWC Docs – Base Components](https://developer.salesforce.com/docs/component-library/overview/components) | [lightning-record-form](https://developer.salesforce.com/docs/component-library/bundle/lightning-record-form/documentation) | [lightning-record-edit-form](https://developer.salesforce.com/docs/component-library/bundle/lightning-record-edit-form/documentation) | [lightning-record-view-form](https://developer.salesforce.com/docs/component-library/bundle/lightning-record-view-form/documentation) | [LDS](https://developer.salesforce.com/docs/atlas.en-us.lwc.meta/lwc/data_service.htm)*
