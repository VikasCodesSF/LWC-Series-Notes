# ⚡ LWC – Apex in LWC

## 1. Introduction – Apex in LWC

There are **3 ways** to interact with Salesforce data in LWC:

| Approach | Ease | Flexibility |
|---|---|---|
| Base Lightning Components (LDS) | Easiest | Less flexible |
| LDS Wire Adapters & Functions | Easy | More flexible |
| **Apex** | Complex | **Extremely Flexible** |

> Use Apex when Base Components and LDS Wire Adapters are not sufficient.

### When to Use Apex?
- To work with objects **not supported by UI API** (e.g., Task, Event)
- To work with operations UI API doesn't support (e.g., load 200 Accounts where Amount > $1M)
- To perform **transactional operations** (e.g., create Account + Opportunity atomically — if one fails, both rollback)
- To call a method that performs **DML** (insert, update, delete) — these cannot be `cacheable=true`
- To control **when** the invocation occurs (on button click, on condition, etc.)
- To work with objects from an **ES6 module** that doesn't extend `LightningElement`

### Two Ways to Call Apex in LWC

```
Apex in LWC
├── Wire Apex Methods      (automatic, reactive)
└── Call Apex Imperatively (manual, on-demand)
```

---

## 2. Expose Apex Methods to LWC

### Requirements
1. The Apex method must be **`static`** and either **`global`** or **`public`**
2. The method must be annotated with **`@AuraEnabled`**

### Syntax
```apex
public with sharing class AccountController {
    @AuraEnabled(cacheable=true)
    public static List<Account> getAccountList() {
        return [SELECT Id, Name, Type, Industry FROM Account LIMIT 5];
    }
}
```

### `@AuraEnabled` Key Rules

| Scenario | `cacheable` Setting |
|---|---|
| Read-only (SELECT queries only) | `@AuraEnabled(cacheable=true)` |
| DML operations (insert/update/delete) | `@AuraEnabled` (no cacheable) |

> **Important:** Methods with DML (insert, update, delete) **cannot** use `cacheable=true`. Only use `cacheable=true` for read-only methods.

---

## 3. Import Apex Methods

Use the `@salesforce/apex` scoped package to import Apex methods into LWC.

### Syntax
```javascript
import apexMethodName from '@salesforce/apex/Namespace.Classname.apexMethodReference';
```

### Import Parameters
| Parameter | Description |
|---|---|
| `apexMethodName` | Symbol identifying the Apex method in JS |
| `apexMethodReference` | The name of the Apex method to import |
| `Classname` | The name of the Apex class |
| `Namespace` | Omit if same namespace as component. Include if in a managed package. |

### Example
```javascript
import getAccountList from '@salesforce/apex/AccountController.getAccountList';
```

---

## 4. Wire Apex Methods

### Overview
- Use `@wire` to **reactively call** an Apex method.
- LWC calls the method automatically and provides the result as reactive data.
- Works only with methods annotated `@AuraEnabled(cacheable=true)`.
- Can wire to a **property** (simple) or a **function** (for data transformation).

### Syntax
```javascript
import apexMethodName from '@salesforce/apex/Namespace.Classname.apexMethodReference';
@wire(apexMethodName, { apexMethodParams })
propertyOrFunction;
```

> **Important Note:** If a parameter value is **`null`**, the method IS called. If the value is **`undefined`**, the method is NOT called.

### 4a. Wire to a Property

Best for: Simple display of data with no transformation needed.

**Apex Class:**
```apex
public with sharing class AccountController {
    @AuraEnabled(cacheable=true)
    public static List<Account> getAccountList() {
        return [SELECT Id, Name, Type, Industry FROM Account LIMIT 5];
    }
}
```

**JS:**
```javascript
import { LightningElement, wire } from 'lwc';
import getAccountList from '@salesforce/apex/AccountController.getAccountList';

export default class ApexWireDemo extends LightningElement {
    @wire(getAccountList)
    accounts;   // accounts.data = array of records, accounts.error = error object
}
```

**HTML:**
```html
<template>
    <lightning-card title="Apex Wire To Property Demo">
        <div class="slds-p-around_medium">
            <template lwc:if={accounts.data}>
                <template for:each={accounts.data} for:item="account">
                    <div class="slds-box slds-box_xx-small" key={account.Id}>
                        <p><strong>Name :</strong> {account.Name}</p>
                        <p><strong>Type :</strong> {account.Type}</p>
                        <p><strong>Industry :</strong> {account.Industry}</p>
                    </div>
                </template>
            </template>
        </div>
    </lightning-card>
</template>
```

> **Note:** Data comes in array format from Apex, so use `for:each` loop inside `lwc:if`.

### 4b. Wire to a Function

Best for: **Transforming/processing data** before displaying it.

**JS:**
```javascript
import { LightningElement, wire } from 'lwc';
import getAccountList from '@salesforce/apex/AccountController.getAccountList';

export default class ApexWireDemo extends LightningElement {
    accountList;

    @wire(getAccountList)
    accountsHandler({ data, error }) {
        if (data) {
            // Transform: replace full type strings with short labels
            this.accountList = data.map(item => {
                let newType = item.Type === 'Customer - Channel' ? 'Channel'
                            : item.Type === 'Customer - Direct'  ? 'Direct' : '-------';
                return { ...item, newType };
            });
        }
        if (error) {
            console.error(error);
        }
    }
}
```

**HTML (uses `accountList` directly, not `accountList.data`):**
```html
<template lwc:if={accountList}>
    <template for:each={accountList} for:item="account">
        <div class="slds-box slds-box_xx-small" key={account.Id}>
            <p><strong>Name : </strong>{account.Name}</p>
            <p><strong>Type : </strong>{account.newType}</p>
            <p><strong>Industry : </strong>{account.Industry}</p>
        </div>
    </template>
</template>
```

### Wire to Property vs Wire to Function — Comparison

| Aspect | Wire to Property | Wire to Function |
|---|---|---|
| Data access | `accounts.data` | Direct variable (e.g., `accountList`) |
| Error access | `accounts.error` | Via function parameter `error` |
| Data transform | Not possible directly | ✅ Yes, transform before assigning |
| Use case | Simple display | Transform/process data |
| Syntax | `@wire(method) propName;` | `@wire(method) handlerFn({data, error}){}` |

---

## 5. Wire Apex Method with Parameters

Pass dynamic parameters to a wired Apex method. The wire re-executes automatically whenever the parameter value changes (reactive).

### Apex Method (with parameter)
```apex
@AuraEnabled(cacheable=true)
public static List<Account> filterAccountTypeType(String type) {
    return [SELECT Id, Name, Type, Industry FROM Account WHERE Type = :type LIMIT 5];
}
```

### JS
```javascript
import { LightningElement, wire } from 'lwc';
import filterAccountTypeType from '@salesforce/apex/AccountController.filterAccountTypeType';

export default class WireApexWithParam extends LightningElement {
    selectedType = '';

    @wire(filterAccountTypeType, { type: '$selectedType' })  // $ makes it reactive
    filteredAccounts;

    get typeOptions() {
        return [
            { label: 'Customer - Direct',  value: 'Customer - Direct' },
            { label: 'Customer - Channel', value: 'Customer - Channel' }
        ];
    }

    typeHandler(event) {
        this.selectedType = event.target.value;
    }
}
```

**HTML:**
```html
<template>
    <lightning-card title="Apex Wire with Parameter">
        <div class="slds-p-around_medium">
            <lightning-combobox
                name="type"
                label="Choose your Type"
                value={selectedType}
                options={typeOptions}
                onchange={typeHandler}>
            </lightning-combobox>
            <template lwc:if={filteredAccounts.data}>
                <template for:each={filteredAccounts.data} for:item="account">
                    <div class="slds-box slds-box_xx-small" key={account.Id}>
                        <p><strong>Name :</strong> {account.Name}</p>
                        <p><strong>Type :</strong> {account.Type}</p>
                    </div>
                </template>
            </template>
        </div>
    </lightning-card>
</template>
```

> **`$` prefix makes a parameter reactive.** When `selectedType` changes, `@wire` re-runs automatically.

---

## 6. Call Apex Methods Imperatively

### When to Use Imperative over @wire

| Use Imperative When... |
|---|
| Method is NOT annotated with `cacheable=true` (e.g., DML methods) |
| You need to control **when** the invocation happens (e.g., on button click) |
| Working with objects not supported by UI API (Task, Event) |
| Calling from an ES6 module that doesn't extend `LightningElement` |

### Syntax
```javascript
apexMethodName(params)
    .then(result => { /* handle result */ })
    .catch(error => { /* handle error */ });
```

The imported Apex method returns a **Promise**.

### Example: Basic Imperative Call (on Button Click)

**Apex:**
```apex
@AuraEnabled(cacheable=true)
public static List<Account> getAccountList() {
    return [SELECT Id, Name, Type, Industry FROM Account LIMIT 5];
}
```

**JS:**
```javascript
import { LightningElement } from 'lwc';
import getAccountList from '@salesforce/apex/AccountController.getAccountList';

export default class ApexImperativeDemo extends LightningElement {
    accounts;

    handleClick() {
        getAccountList()
            .then(result => {
                console.log(result);
                this.accounts = result;  // result is an array of objects
            })
            .catch(error => {
                console.error(error);
            });
    }
}
```

**HTML:**
```html
<template>
    <lightning-card title="Apex Imperative Demo">
        <div class="slds-p-around_medium">
            <lightning-button variant="brand" label="Call Account Method"
                onclick={handleClick}></lightning-button>
            <template lwc:if={accounts}>
                <template for:each={accounts} for:item="account">
                    <div class="slds-box slds-box_xx-small" key={account.Id}>
                        <p>Account Name: {account.Name}</p>
                    </div>
                </template>
            </template>
        </div>
    </lightning-card>
</template>
```

### Example: Imperative with Parameters

**Apex:**
```apex
@AuraEnabled(cacheable=true)
public static List<Account> findAccounts(String searchKey) {
    String key = '%' + searchKey + '%';
    return [SELECT Id, Name, Type, Industry FROM Account WHERE Name LIKE :key LIMIT 5];
}
```

**JS (with Debouncing):**
```javascript
import { LightningElement } from 'lwc';
import findAccounts from '@salesforce/apex/AccountController.findAccounts';

export default class ApexImperativeWithParamsDemo extends LightningElement {
    searchKey = '';
    accounts;
    timer;  // for debouncing

    searchHandler(event) {
        window.clearTimeout(this.timer);   // clear previous timer
        this.searchKey = event.target.value;
        this.timer = setTimeout(() => {
            this.callApex();
        }, 2000);  // wait 2s after user stops typing
    }

    callApex() {
        findAccounts({ searchKey: this.searchKey })
            .then(result => {
                this.accounts = result;
            })
            .catch(error => {
                console.error(error);
            });
    }
}
```

**HTML:**
```html
<template>
    <lightning-card title="Apex Imperative with Params Demo">
        <div class="slds-p-around_medium">
            <lightning-input
                type="search"
                onchange={searchHandler}
                label="Search Account"
                value={searchKey}>
            </lightning-input>
            <template lwc:if={accounts}>
                <template for:each={accounts} for:item="account">
                    <div class="slds-box slds-box_xx-small" key={account.Id}>
                        <p>Name - {account.Name}</p>
                        <p>Type - {account.Type}</p>
                        <p>Industry - {account.Name}</p>
                    </div>
                </template>
            </template>
        </div>
    </lightning-card>
</template>
```

> **Debouncing** prevents excessive Apex calls while the user is still typing. `clearTimeout` + `setTimeout` pattern delays the call by 2 seconds after the last keystroke.

---

## 7. Refresh Apex (refreshApex)

### Problem
After performing a DML operation (update/insert), the data displayed by `@wire` is still **showing old cached data** because `@wire` uses LDS caching. We need a way to **re-fetch data without a full page refresh**.

### Solution: `refreshApex`
Import `refreshApex` from `@salesforce/apex` and call it with the **wired property** to force a re-fetch.

> **Note:** `refreshApex` only works with **`@wire`** (property syntax). In the **imperative** approach, just call the Apex method again — no `refreshApex` needed.

### Complete Demo — DataTable with Inline Editing + Update + Refresh

This is the full working example combining:
- Fetching contacts via `@wire`
- Displaying in `lightning-datatable` with inline editing
- Updating records using `updateRecord` from `lightning/uiRecordApi`
- Refreshing wire data without page reload using `refreshApex`

#### Step 1: Apex Class
```apex
public with sharing class refreshContactController {
    @AuraEnabled(cacheable=true)
    public static List<Contact> getContactList() {
        return [SELECT Id, FirstName, LastName, Email FROM Contact LIMIT 10];
    }
}
```

#### Step 2: JS
```javascript
import { LightningElement, wire }    from 'lwc';
import getContactList               from '@salesforce/apex/refreshContactController.getContactList';
import { updateRecord }             from 'lightning/uiRecordApi';
import { ShowToastEvent }           from 'lightning/platformShowToastEvent';
import { refreshApex }              from '@salesforce/apex';

// Column definitions for the datatable (constant — never changes)
const columns = [
    { label: 'First Name', fieldName: 'FirstName', editable: true },
    { label: 'Last Name',  fieldName: 'LastName',  editable: true },
    { label: 'Email',      fieldName: 'Email',     type: 'email'  }
];

export default class RefreshDemoLwc extends LightningElement {
    columns      = columns;
    draftValues  = [];   // stores inline-edited values (shown in yellow)

    // Wire to property — needed so refreshApex can work
    @wire(getContactList)
    contact;

    // Getter to check if contacts are available
    get isContactAvaibale() {
        console.log(JSON.stringify(this.contact));
        return this.contact && this.contact.data && this.contact.data.length > 0
               ? 'Yes' : 'NO';
    }

    handleSave(event) {
        console.log(event.detail.draftValues);

        // 1. Build array of record inputs from draft values
        //    slice() creates a shallow copy (immutable), map() loops over each draft
        //    Object.assign({}, draft) creates a new object per draft
        //    return {fields} — updateRecord needs a {fields} object
        const recordInputs = event.detail.draftValues.slice().map(draft => {
            const fields = Object.assign({}, draft);
            return { fields };
        });

        // 2. updateRecord() accepts only one record at a time, so map to promises
        const promises = recordInputs.map(recordInput => updateRecord(recordInput));

        // 3. Promise.all ensures ALL updates succeed before proceeding
        Promise.all(promises)
            .then(result => {
                this.showToastMsg('Success', 'Contacts updated');
                this.draftValues = [];             // clear yellow highlight
                return refreshApex(this.contact);  // re-fetch wire data
            })
            .catch(error => {
                this.showToastMsg('Error creating record', error.body.message, error);
            });
    }

    showToastMsg(title, message, variant) {
        this.dispatchEvent(
            new ShowToastEvent({
                title:   title,
                message: message,
                variant: variant || 'success'
            })
        );
    }
}
```

#### Step 3: HTML
```html
<template>
    <lightning-card title="Refresh Apex Demo" icon-name="custom:custom63">
        <div class="slds-m-around_medium">
            <template lwc:if={contact.data}>
                <lightning-datatable
                    key-field="Id"
                    data={contact.data}
                    columns={columns}
                    onsave={handleSave}
                    draft-values={draftValues}>
                </lightning-datatable>
            </template>
        </div>
    </lightning-card>
</template>
```

### Wire as Function — Using refreshApex with contactResult

When wiring to a function, you need to store the raw wire result separately to use `refreshApex`:

```javascript
contactResult;  // stores the raw wire result

@wire(getContactList)
contact(result) {
    this.contactResult = result;   // save for refreshApex
    const { data, value } = result;
    // process data...
}

// Then refresh using the stored raw result:
return refreshApex(this.contactResult);
```

---

## 8. Wire vs Imperative — Complete Comparison

| Aspect | @wire | Imperative |
|---|---|---|
| Trigger | Automatic (reactive) | Manual (on demand) |
| `cacheable=true` required | Yes | No (works for DML too) |
| DML support | No | Yes |
| Parameter reactivity | Yes (with `$` prefix) | Manual re-call |
| Data access | `prop.data` / `prop.error` | `.then()` / `.catch()` |
| refreshApex needed | Yes (after external DML) | No — just re-call method |
| Best for | Auto-loading read data | Button-triggered calls, DML |

---

## 9. Key Concepts — Quick Reference

### `updateRecord` from `lightning/uiRecordApi`
- Accepts a single record input object: `{ fields: { Id: ..., FieldName: ... } }`
- Returns a **Promise**
- Import: `import { updateRecord } from 'lightning/uiRecordApi';`

### `Promise.all()`
- Takes an array of Promises
- Resolves only when **ALL** promises resolve
- Rejects immediately if **ANY** promise fails
- Used here to update multiple records in parallel and wait for all to finish

### Debouncing
- Technique to delay execution until user stops typing
- Pattern: `clearTimeout(this.timer)` → `this.timer = setTimeout(() => { ... }, 2000)`
- Prevents excessive server calls on every keystroke

### `slice()` + `map()` + `Object.assign({})` Pattern (in handleSave)
```javascript
const recordInputs = event.detail.draftValues
    .slice()                         // shallow copy (immutable)
    .map(draft => {
        const fields = Object.assign({}, draft);  // new object per draft
        return { fields };                         // updateRecord expects {fields}
    });
```

### `draft-values` in `lightning-datatable`
- Stores the values the user has edited inline (shown in yellow)
- After a successful save, reset it: `this.draftValues = []`
- This removes the yellow highlight from edited cells

---

## 10. Important Tips & Gotchas

- Use `@AuraEnabled(cacheable=true)` **only for read-only** methods. Any DML method must NOT have `cacheable=true`.
- `@wire` always returns `{ data, error }`. Always check `data` before rendering.
- **`undefined` vs `null` in wire params:** `null` → method called; `undefined` → method NOT called.
- `refreshApex` only works with **wire to property**. For wire to function, store the result in a variable and pass that to `refreshApex`.
- In the **imperative** approach, you do NOT need `refreshApex`. Simply call the Apex method again after DML to get fresh data.
- The `$` prefix in `@wire` parameters makes them **reactive** — any change to the property automatically re-calls the wired method.
- `Promise.all` fails fast — if any single promise rejects, the entire `.catch` block runs.
- Always use `with sharing` in Apex classes used by LWC to enforce sharing rules.

---

*📌 References: [Salesforce Apex in LWC Docs](https://developer.salesforce.com/docs/atlas.en-us.lwc.meta/lwc/apex.htm) | [Wire Apex](https://developer.salesforce.com/docs/atlas.en-us.lwc.meta/lwc/apex_wire_method.htm) | [Imperative Apex](https://developer.salesforce.com/docs/atlas.en-us.lwc.meta/lwc/apex_call_imperative.htm) | [refreshApex](https://developer.salesforce.com/docs/atlas.en-us.lwc.meta/lwc/apex_result_caching.htm) | [uiRecordApi](https://developer.salesforce.com/docs/atlas.en-us.lwc.meta/lwc/reference_lightning_ui_api_record.htm)*