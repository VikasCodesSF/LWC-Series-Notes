# LWC Wire Adapters — Interview Cheat Sheet
---

## How to Answer "What are wire adapters?"

> Wire adapters are Salesforce-provided modules that let LWC read Salesforce data reactively via `@wire`, without Apex. They're grouped by module: `uiRecordApi` (records), `uiObjectInfoApi` (object metadata), `uiRelatedListApi` (related lists), plus `uiListsApi`, `uiAppsApi`, `uiGraphQLApi`, and custom Apex wires. All return `{data, error}`, and reactive params use the `$` prefix.

---

# 📗 SHEET 1: `lightning/uiRecordApi`

**Purpose:** Read, create, update, delete a single record — no Apex needed.

```javascript
import {
  getRecord, getRecords, getFieldValue, getFieldDisplayValue,
  getRecordCreateDefaults, createRecord, updateRecord, deleteRecord
} from 'lightning/uiRecordApi';
```

### `getRecord` — fetch one record

```javascript
import { LightningElement, api, wire } from 'lwc';
import { getRecord, getFieldValue } from 'lightning/uiRecordApi';
import NAME_FIELD from '@salesforce/schema/Account.Name';

const FIELDS = [NAME_FIELD];

export default class Demo extends LightningElement {
  @api recordId;

  @wire(getRecord, { recordId: '$recordId', fields: FIELDS })
  account;

  get name() {
    return getFieldValue(this.account.data, NAME_FIELD);
  }
}
```
> `record.data.fields.Name.value` — always nested. `$recordId` = reactive.

### `getFieldDisplayValue` — formatted value

```javascript
get formattedRevenue() {
  return getFieldDisplayValue(this.account.data, REVENUE_FIELD);
}
// raw: 350000000 → display: "$350,000,000"
```

### `getRecords` — batch, multiple objects at once

```javascript
@wire(getRecords, {
  records: [
    { recordIds: ['005...'], fields: [USER_NAME_FIELD] },
    { recordIds: ['001...'], fields: [ACCOUNT_NAME_FIELD] }
  ]
})
wiredRecords;
```

### `createRecord` — imperative, NOT `@wire`

```javascript
import { createRecord } from 'lightning/uiRecordApi';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';
import NAME_FIELD from '@salesforce/schema/Account.Name';

async createAccount() {
  const fields = { [NAME_FIELD.fieldApiName]: this.name };
  const recordInput = { apiName: ACCOUNT_OBJECT.objectApiName, fields };
  const account = await createRecord(recordInput);
  this.accountId = account.id;
}
```

### `updateRecord` — must include Id

```javascript
import { updateRecord } from 'lightning/uiRecordApi';
import ID_FIELD from '@salesforce/schema/Account.Id';

updateAccount() {
  const fields = { [ID_FIELD.fieldApiName]: this.recordId, Name: this.name };
  updateRecord({ fields });
}
```

### `deleteRecord`

```javascript
import { deleteRecord } from 'lightning/uiRecordApi';
deleteRecord(this.recordId);
```

**🎯 Interview one-liner:** *"`@wire` is for reads; create/update/delete are imperative functions because writes need explicit developer control over timing — that's a direct Trailhead rule."*

---

# 📘 SHEET 2: `lightning/uiObjectInfoApi`

**Purpose:** Read object *metadata* — fields, Record Types, picklists — not record data.

```javascript
import {
  getObjectInfo, getObjectInfos,
  getPicklistValues, getPicklistValuesByRecordType
} from 'lightning/uiObjectInfoApi';
```

### `getObjectInfo` — metadata for one object

```javascript
import { getObjectInfo } from 'lightning/uiObjectInfoApi';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';

@wire(getObjectInfo, { objectApiName: ACCOUNT_OBJECT })
objectInfo;

get defaultRecordTypeId() {
  return this.objectInfo.data?.defaultRecordTypeId;
}
```
> Returns: `fields`, `recordTypeInfos`, `defaultRecordTypeId`, `childRelationships`, `createable/updateable/deletable`.

### `getObjectInfos` — batch, multiple objects

```javascript
@wire(getObjectInfos, { objectApiNames: [ACCOUNT_OBJECT, CONTACT_OBJECT] })
multiInfo;
// result: data.results[] — same order as requested
```

### `getPicklistValues` — THE classic combo with `getObjectInfo`

```javascript
import { getObjectInfo, getPicklistValues } from 'lightning/uiObjectInfoApi';
import INDUSTRY_FIELD from '@salesforce/schema/Account.Industry';

@wire(getObjectInfo, { objectApiName: ACCOUNT_OBJECT })
objectInfo;

@wire(getPicklistValues, {
  recordTypeId: '$objectInfo.data.defaultRecordTypeId',  // chained!
  fieldApiName: INDUSTRY_FIELD
})
industryValues;

get options() {
  return this.industryValues.data?.values ?? [];
}
```

### `getPicklistValuesByRecordType` — ALL picklists for a Record Type, one call

```javascript
@wire(getPicklistValuesByRecordType, {
  objectApiName: 'Account',
  recordTypeId: '$objectInfo.data.defaultRecordTypeId'
})
allPicklists;
```

**🎯 Interview one-liner:** *"`getObjectInfo` gives you the Record Type Id, `getPicklistValues` needs that Id as input — it's the most common chained-wire pattern in real Salesforce dev."*

---

# 📙 SHEET 3: `lightning/uiRelatedListApi`

**Purpose:** Related lists (Contacts, Opportunities under an Account) — no Apex.

```javascript
import {
  getRelatedListRecords, getRelatedListCount,
  getRelatedListInfo, getRelatedListsInfo,
  getRelatedListRecordsBatch, getRelatedListInfoBatch
} from 'lightning/uiRelatedListApi';
```

### `getRelatedListRecords` — the main one

```javascript
import { getRelatedListRecords } from 'lightning/uiRelatedListApi';

@wire(getRelatedListRecords, {
  parentRecordId: '$recordId',
  relatedListId: 'Contacts',
  fields: ['Contact.Name', 'Contact.Email'],
  sortBy: ['-Contact.CreatedDate'],   // "-" = descending
  pageSize: 5
})
wiredContacts({ data, error }) {
  if (data) this.records = data.records;
}
```

### `getRelatedListCount` — just the number

```javascript
@wire(getRelatedListCount, { parentRecordId: '$recordId', relatedListId: 'Contacts' })
countResult;

get count() { return this.countResult.data?.count; }
```

### `getRelatedListInfo` — metadata (uses `parentObjectApiName`, not recordId!)

```javascript
import ACCOUNT_OBJECT from '@salesforce/schema/Account';

@wire(getRelatedListInfo, {
  parentObjectApiName: ACCOUNT_OBJECT,   // OBJECT api name, not a record Id
  relatedListId: 'Contacts'
})
relatedListInfo;
```

### `getRelatedListsInfo` — ALL related lists on the layout, one call

```javascript
@wire(getRelatedListsInfo, { parentObjectApiName: ACCOUNT_OBJECT })
allRelatedLists;
```

### Batch versions

```javascript
@wire(getRelatedListRecordsBatch, {
  parentRecordId: '$recordId',
  relatedListParameters: [
    { relatedListId: 'Contacts', fields: ['Contact.Name'] },
    { relatedListId: 'Opportunities', fields: ['Opportunity.Name'] }
  ]
})
batchResult;
// each result independent: data.results[0].result / data.results[1].result
```

**🎯 Interview one-liner:** *"`parentRecordId` for data adapters, `parentObjectApiName` for metadata adapters — and a related list missing from the page layout returns an error, not empty data, in a batch call."*

---

## ⚡ Cross-Module Rules That Apply Everywhere

| Rule | Why |
|---|---|
| `$` prefix = reactive parameter | Re-fires the wire automatically when that value changes |
| Response = `{data, error}` | Never assume `data` exists — always guard with `lwc:if` |
| Never use `@wire` for DML | `createRecord`/`updateRecord`/`deleteRecord`/`getRelatedList*` writes are all imperative |
| Schema imports > hardcoded strings | `@salesforce/schema/Account.Name` catches typos at build time |
| `fields` vs `optionalFields` | `fields` fails the whole call if inaccessible; `optionalFields` silently omits |
| Chained wires need `$parent.data.x` | e.g. `getPicklistValues` waits on `$objectInfo.data.defaultRecordTypeId` |

---
