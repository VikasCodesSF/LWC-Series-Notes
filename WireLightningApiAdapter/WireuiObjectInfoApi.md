# LWC Wire Adapters — `lightning/uiObjectInfoApi`
> Module: `lightning/uiObjectInfoApi` | Short & Crisp Reference

---

## 📦 Import Module

```js
import { getObjectInfo, getObjectInfos, getPicklistValues, getPicklistValuesByRecordType }
  from 'lightning/uiObjectInfoApi';
```

---

## 1️⃣ `getObjectInfo`

**Purpose:** Get metadata of a **single** Salesforce object (fields, record types, relationships, theme).

### Syntax
```js
import { LightningElement, wire } from 'lwc';
import { getObjectInfo } from 'lightning/uiObjectInfoApi';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';

export default class Example extends LightningElement {
    @wire(getObjectInfo, { objectApiName: ACCOUNT_OBJECT })
    objectInfo;
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `objectApiName` | String / ObjectId | ✅ Yes | API name of the object |

### Key Response Properties
| Property | Description |
|---|---|
| `data.apiName` | Object API name |
| `data.label` | Object label |
| `data.fields` | Map of field API names → field metadata |
| `data.recordTypeInfos` | Map of record type IDs → record type info |
| `data.defaultRecordTypeId` | Default record type ID (or master RT: `012000000000000AAA`) |
| `data.childRelationships` | Child relationship metadata |
| `error` | Error object if call fails |

### Key Notes
- Uses **UI API** under the hood
- Use `data.defaultRecordTypeId` as input to `getPicklistValues`
- Use `getObjectInfos` instead when working with **multiple objects**

---

## 2️⃣ `getObjectInfos`

**Purpose:** Get metadata for **multiple** Salesforce objects in a **single** wire call.

### Syntax
```js
import { LightningElement, wire, api } from 'lwc';
import { getObjectInfos } from 'lightning/uiObjectInfoApi';

export default class Example extends LightningElement {
    @api objectApiNames; // e.g., ['Account', 'Contact']

    @wire(getObjectInfos, { objectApiNames: '$objectApiNames' })
    objectInfos;
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `objectApiNames` | String[] | ✅ Yes | Array of object API names |

### Key Response Properties
| Property | Description |
|---|---|
| `data.results` | Ordered array of result objects (same order as input) |
| `data.results[i].result` | Object Info metadata for that object |
| `data.results[i].statusCode` | HTTP status code (200 = success) |
| `error` | Returned **only** if the entire server call fails |

### Key Notes
- Individual object errors appear in `data.results[].result`, **not** in `error`
- Results are returned in the **same order** as `objectApiNames`
- Use `getObjectInfo` if you only need **one** object

---

## 3️⃣ `getPicklistValues`

**Purpose:** Get picklist values for a **specific field** of a specific record type.

### Syntax
```js
import { LightningElement, wire } from 'lwc';
import { getPicklistValues } from 'lightning/uiObjectInfoApi';
import INDUSTRY_FIELD from '@salesforce/schema/Account.Industry';

export default class Example extends LightningElement {
    recordTypeId = '012000000000000AAA'; // or from getObjectInfo

    @wire(getPicklistValues, {
        recordTypeId: '$recordTypeId',
        fieldApiName: INDUSTRY_FIELD
    })
    picklistValues;
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `recordTypeId` | String | ✅ Yes | Record type ID. Use `defaultRecordTypeId` from `getObjectInfo` |
| `fieldApiName` | FieldId | ✅ Yes | Schema reference e.g. `Account.Industry` |

### Key Response Properties
| Property | Description |
|---|---|
| `data.values` | Array of `{ label, value }` objects |
| `data.defaultValue` | Default picklist value (if set) |
| `data.controllerValues` | Map used for dependent picklists |
| `error` | Error object if call fails |

### Key Notes
- Both params are **required**
- Use `getObjectInfo` first to obtain `recordTypeId` dynamically
- Use `getPicklistValuesByRecordType` if you need **all picklists** of a record type
- Master record type ID: `012000000000000AAA`

### Common Pattern (chained wires)
```js
@wire(getObjectInfo, { objectApiName: ACCOUNT_OBJECT })
wiredObjectInfo;

get recordTypeId() {
    return this.wiredObjectInfo.data?.defaultRecordTypeId;
}

@wire(getPicklistValues, {
    recordTypeId: '$recordTypeId',
    fieldApiName: INDUSTRY_FIELD
})
picklistValues;
```

---

## 4️⃣ `getPicklistValuesByRecordType`

**Purpose:** Get **all picklist values** for **all picklist fields** of a specified record type in one call.

### Syntax
```js
import { LightningElement, wire, api } from 'lwc';
import { getPicklistValuesByRecordType } from 'lightning/uiObjectInfoApi';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';

export default class Example extends LightningElement {
    @api recordTypeId;

    @wire(getPicklistValuesByRecordType, {
        objectApiName: ACCOUNT_OBJECT,
        recordTypeId: '$recordTypeId'
    })
    picklistValues;

    get industryOptions() {
        return this.picklistValues.data?.picklistFieldValues?.Industry?.values ?? [];
    }
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `objectApiName` | String / ObjectId | ✅ Yes | API name of the object |
| `recordTypeId` | String | ✅ Yes | Record type ID |

### Key Response Properties
| Property | Description |
|---|---|
| `data.picklistFieldValues` | Map of field API name → picklist data |
| `data.picklistFieldValues.FieldName.values` | Array of `{ label, value }` |
| `data.picklistFieldValues.FieldName.defaultValue` | Default value for that field |
| `error` | Error object if call fails |

### Key Notes
- Returns **all** picklist fields in one response (more efficient than multiple `getPicklistValues` calls)
- Use `getPicklistValues` when you only need **one specific field**
- Picklist values are **scoped to** the record type

---

## ⚡ Quick Comparison

| Adapter | Use When | Scope |
|---|---|---|
| `getObjectInfo` | Need metadata for 1 object | Single object |
| `getObjectInfos` | Need metadata for multiple objects | Multiple objects |
| `getPicklistValues` | Need picklist values for 1 field | Single field |
| `getPicklistValuesByRecordType` | Need all picklist values for a record type | All fields in a record type |

---

## 🔗 LWC Recipes Reference
- `c-wire-get-object-info`
- `c-wire-get-picklist-values`
- `c-wire-get-picklist-values-by-record-type`

> All adapters are part of `lightning/uiObjectInfoApi` module and are **reactive** — prefix reactive variables with `$` in wire parameters.