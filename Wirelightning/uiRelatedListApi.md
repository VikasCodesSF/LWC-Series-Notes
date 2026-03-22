# LWC Wire Adapters — `lightning/uiRelatedListApi`
> Module: `lightning/uiRelatedListApi` | Short & Crisp Reference | Introduced: Spring/Summer '22

---

## 📦 Import Module

```js
import {
  getRelatedListCount,
  getRelatedListInfo,
  getRelatedListInfoBatch,
  getRelatedListRecords,
  getRelatedListRecordsBatch,
  getRelatedListsInfo
} from 'lightning/uiRelatedListApi';
```

> Built on **Lightning Data Service (LDS)** and **UI API** — no Apex needed!

---

## 1️⃣ `getRelatedListCount`

**Purpose:** Get the **record count** of a specific related list.

### Syntax
```js
import { LightningElement, api, wire } from 'lwc';
import { getRelatedListCount } from 'lightning/uiRelatedListApi';

export default class Example extends LightningElement {
    @api recordId;

    @wire(getRelatedListCount, {
        parentRecordId: '$recordId',
        relatedListId: 'Contacts'
    })
    relatedListCount;
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `parentRecordId` | String | ✅ Yes | ID of the parent record (e.g. Account ID) |
| `relatedListId` | String | ✅ Yes | API name of the related list (e.g. `Contacts`, `Cases`) |

### Key Response Properties
| Property | Description |
|---|---|
| `data.count` | Number of records in the related list |
| `error` | Error object if call fails |

### Key Notes
- Lightweight — use for **badge/count display** without loading full records
- Reactive — re-fetches when `parentRecordId` changes

---

## 2️⃣ `getRelatedListInfo`

**Purpose:** Get **metadata** for a **single** related list (columns, label, sort, etc.).

### Syntax
```js
import { LightningElement, api, wire } from 'lwc';
import { getRelatedListInfo } from 'lightning/uiRelatedListApi';

export default class Example extends LightningElement {
    @api recordId;

    @wire(getRelatedListInfo, {
        parentObjectApiName: 'Account',
        relatedListId: 'Contacts',
        // recordTypeId: 'optional',
        // fields: ['Contact.Name'],
        // restrictColumnsToLayout: false
    })
    relatedListInfo;
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `parentObjectApiName` | String | ✅ Yes | API name of the parent object |
| `relatedListId` | String | ✅ Yes | API name of the related list |
| `recordTypeId` | String | ❌ Optional | Parent record type ID |
| `fields` | String[] | ❌ Optional | Field API names to include |
| `optionalFields` | String[] | ❌ Optional | Additional fields to include |
| `restrictColumnsToLayout` | Boolean | ❌ Optional | `true` = return only layout columns |

### Key Response Properties
| Property | Description |
|---|---|
| `data.label` | Related list label |
| `data.columns` | Array of column metadata |
| `data.fields` | Array of field metadata |
| `data.sortBy` | Default sort field |
| `data.relatedListId` | Related list API name |
| `error` | Error object if call fails |

### Key Notes
- Use for building **custom column headers** or dynamic field rendering
- Use `getRelatedListInfoBatch` for multiple related lists

---

## 3️⃣ `getRelatedListInfoBatch`

**Purpose:** Get metadata for **multiple related lists** in a **single wire call**.

### Syntax
```js
import { LightningElement, api, wire } from 'lwc';
import { getRelatedListInfoBatch } from 'lightning/uiRelatedListApi';

export default class Example extends LightningElement {
    @api recordId;

    @wire(getRelatedListInfoBatch, {
        parentObjectApiName: 'Account',
        relatedListParameters: [
            { relatedListId: 'Contacts' },
            { relatedListId: 'Opportunities' }
        ]
    })
    batchInfo;
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `parentObjectApiName` | String | ✅ Yes | API name of the parent object |
| `relatedListParameters` | Object[] | ✅ Yes | Array of `{ relatedListId, fields?, optionalFields? }` |
| `recordTypeId` | String | ❌ Optional | Parent record type ID |

### Key Response Properties
| Property | Description |
|---|---|
| `data.results` | Ordered array of results (same order as input) |
| `data.results[i].result` | Metadata for that related list |
| `data.results[i].statusCode` | HTTP status (200 = success) |
| `error` | Returned only if entire server call fails |

### Key Notes
- Individual errors are in `data.results[i].result`, **not** in `error`
- More efficient than multiple `getRelatedListInfo` calls

---

## 4️⃣ `getRelatedListRecords`

**Purpose:** Get **records** from a **single** related list.

### Syntax
```js
import { LightningElement, api, wire } from 'lwc';
import { getRelatedListRecords } from 'lightning/uiRelatedListApi';

export default class Example extends LightningElement {
    @api recordId;

    @wire(getRelatedListRecords, {
        parentRecordId: '$recordId',
        relatedListId: 'Contacts',
        fields: ['Contact.Id', 'Contact.Name', 'Contact.Email'],
        // optionalFields: ['Contact.Phone'],
        // pageSize: 10,
        // sortBy: ['Contact.Name'],
        // where: { Name: { like: "A%" } }
    })
    relatedRecords;

    get records() {
        return this.relatedRecords.data?.records ?? [];
    }
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `parentRecordId` | String | ✅ Yes | ID of the parent record |
| `relatedListId` | String | ✅ Yes | API name of the related list |
| `fields` | String[] | ✅ Yes* | Field API names (format: `Object.Field`) |
| `optionalFields` | String[] | ❌ Optional | Additional fields to return |
| `pageSize` | Number | ❌ Optional | Records per page (default varies) |
| `sortBy` | String[] | ❌ Optional | Fields to sort by (prefix `-` for desc) |
| `where` | Object | ❌ Optional | GraphQL-style filter object |

> *At least one field is required

### Key Response Properties
| Property | Description |
|---|---|
| `data.records` | Array of record objects |
| `data.records[i].fields` | Map of field name → `{ value, displayValue }` |
| `data.listInfoETag` | ETag for cache control |
| `data.pageSize` | Page size used |
| `data.currentPageToken` | Token for current page |
| `data.nextPageToken` | Token for next page (`null` if last page) |
| `error` | Error object if call fails |

### Accessing Field Values
```js
// data.records[0].fields.Name.value
// data.records[0].fields.Name.displayValue
get contactNames() {
    return this.relatedRecords.data?.records.map(r => r.fields.Name.value) ?? [];
}
```

### Key Notes
- **No Apex needed** — built on LDS
- If a related list is NOT on the page layout, it won't return data
- Use `where` for GraphQL-style server-side filtering
- For multiple related lists, use `getRelatedListRecordsBatch`

---

## 5️⃣ `getRelatedListRecordsBatch`

**Purpose:** Get records from **multiple related lists** in a **single wire call**.

### Syntax
```js
import { LightningElement, api, wire } from 'lwc';
import { getRelatedListRecordsBatch } from 'lightning/uiRelatedListApi';

export default class Example extends LightningElement {
    @api recordId;

    @wire(getRelatedListRecordsBatch, {
        parentRecordId: '$recordId',
        relatedListParameters: [
            { relatedListId: 'Contacts', fields: ['Contact.Id', 'Contact.Name'] },
            { relatedListId: 'Opportunities', fields: ['Opportunity.Id', 'Opportunity.Name'] },
            { relatedListId: 'Cases', fields: ['Case.Id', 'Case.CaseNumber'] }
        ]
    })
    batchRecords;

    get allRecords() {
        return this.batchRecords.data?.results ?? [];
    }
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `parentRecordId` | String | ✅ Yes | ID of the parent record |
| `relatedListParameters` | Object[] | ✅ Yes | Array of `{ relatedListId, fields, optionalFields?, pageSize?, sortBy?, where? }` |

### Key Response Properties
| Property | Description |
|---|---|
| `data.results` | Array of result objects (same order as input) |
| `data.results[i].result.records` | Records for that related list |
| `data.results[i].statusCode` | HTTP status (200 = success) |
| `error` | Returned only if entire server call fails |

### Key Notes
- Individual list errors are in `data.results[i]`, **not** in `error`
- If a related list is missing from layout, that result shows an error — others still return
- Most efficient way to load multiple related list records

---

## 6️⃣ `getRelatedListsInfo`

**Purpose:** Get metadata for **all related lists** of an object's **default layout**.

### Syntax
```js
import { LightningElement, wire } from 'lwc';
import { getRelatedListsInfo } from 'lightning/uiRelatedListApi';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';

export default class Example extends LightningElement {
    @wire(getRelatedListsInfo, {
        parentObjectApiName: ACCOUNT_OBJECT,
        // recordTypeId: 'optional'
    })
    relatedListsInfo;

    get relatedLists() {
        return this.relatedListsInfo.data?.relatedLists ?? [];
    }
}
```

### Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|
| `parentObjectApiName` | String / ObjectId | ✅ Yes | API name of the parent object |
| `recordTypeId` | String | ❌ Optional | Parent record type ID |

### Key Response Properties
| Property | Description |
|---|---|
| `data.relatedLists` | Array of related list metadata objects |
| `data.relatedLists[i].relatedListId` | API name (use as input to other adapters) |
| `data.relatedLists[i].label` | Display label |
| `data.relatedLists[i].columns` | Column metadata |
| `error` | Error object if call fails |

### Key Notes
- Returns ALL related lists visible in the default layout — no filtering
- Use `relatedListId` values here as inputs to `getRelatedListRecords`
- Differs from `getRelatedListInfo` (single) and `getRelatedListInfoBatch` (selective multiple)

---

## ⚡ Quick Comparison

| Adapter | Use When | Returns |
|---|---|---|
| `getRelatedListCount` | Show badge/count only | Record count (Number) |
| `getRelatedListInfo` | Metadata for 1 related list | Columns, fields, sort info |
| `getRelatedListInfoBatch` | Metadata for selected multiple lists | Batch metadata results |
| `getRelatedListsInfo` | All related lists of an object | All layout-visible related lists |
| `getRelatedListRecords` | Records from 1 related list | Record data + pagination |
| `getRelatedListRecordsBatch` | Records from multiple lists at once | Batch record results |

---

## 📌 Common Gotchas

- **Layout dependency:** If a related list is NOT added to the parent record's page layout, `getRelatedListRecords` and batch adapters will **not return data** for it
- **Field format:** Always use `Object.Field` format (e.g., `Contact.Name`) in `fields` array
- **Reactive params:** Prefix reactive properties with `$` (e.g., `'$recordId'`)
- **Pagination:** Use `nextPageToken` from response to implement load-more logic
- **No Apex needed:** All adapters use LDS — changes auto-reflect without page refresh

---

## 🔗 LWC Recipe Reference
`c-wire-related-list` — LWC Recipes GitHub repo