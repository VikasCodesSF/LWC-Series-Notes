**BASE LIGHTNING COMPONENTS**

_Working with Salesforce Data in Lightning Web Components_

Published: April 2024 • Salesforce LWC Series

## **Introduction**

Salesforce developers have multiple options when it comes to interacting with org data inside Lightning Web Components. Choosing the right approach for each use case helps you write less code, simpler code, and code that is significantly easier to maintain. The three primary approaches form a spectrum from simplicity to flexibility:

| **Base Lightning Components**<br><br>_Easiest to implement, less flexibility_ | **LDS Wire Adapters & Functions**<br><br>_Easy to use, more flexible than base components_ | **Apex**<br><br>_Extremely flexible, maximum control_ |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------- |

This blog post focuses on the first and most approachable option: Base Lightning Components built on Lightning Data Service (LDS). If you are building a standard form UI for creating, viewing, or editing Salesforce records, these components can save you significant development time.

## **What Is Lightning Data Service?**

Lightning Data Service (LDS) is a centralized data caching framework built on top of the User Interface API (UI API). It acts as an intelligent intermediary between your LWC and the Salesforce server, handling caching, security enforcement, and server optimization automatically.

### **How LDS Works**

**First request:** The LWC asks LDS for a record. LDS checks its client-side cache. On a cache miss, it fetches the record from the server via UI API, saves it to cache, and returns it to the component.

**Subsequent requests:** If the same record is requested again (by any component on the page), LDS serves it from cache instantly - no server call needed.

### **Key Capabilities of LDS**

| **Caching**      | Caches results on the client - reduces server round trips    |
| ---------------- | ------------------------------------------------------------ |
| **Security**     | Respects CRUD access, FLS visibility, and sharing settings   |
| **Metadata**     | Returns field metadata alongside record data in one response |
| **Invalidation** | Automatically invalidates cache on data or metadata changes  |
| **Server calls** | Optimizes and batches server calls for performance           |

**Note on Cache Invalidation**

When Salesforce data or metadata changes, LDS automatically invalidates the relevant cache entry and calls the notifyRecordUpdateAvailable() method to inform wire adapters to refresh their data.

**Why does this matter?** Because Base Lightning Components are built on top of LDS, your components automatically inherit all of these capabilities - caching, security, and optimized server calls - without writing a single line of data-access code.

## **Base Lightning Components Built on LDS**

There are three base lightning components that leverage LDS under the hood. Each is purpose-built for a specific use case and renders Salesforce record data using field metadata, respecting your org's layout configuration, field labels, and help text.

- **lightning-record-form** - Supports create, edit, view, and read-only modes in a single component
- **lightning-record-edit-form** - Displays an editable form for updating or creating records
- **lightning-record-view-form** - Displays a read-only form for viewing record data

### **When Should You Use These Components?**

These components shine in scenarios where you want a metadata-driven UI that mirrors the native Salesforce record page experience. Reach for them when you need to:

- **Create a metadata-driven UI** similar to the record detail page in Salesforce
- **Display record values** based on the field metadata configured in your org
- **Show or hide localized field labels** automatically
- **Display help text** on custom fields without extra markup
- **Perform client-side validation** and enforce validation rules declaratively

## **Component Deep Dive**

### **1\. lightning-record-edit-form**

This component renders an editable form. It is ideal when you want users to create a new record or update an existing one. It does not support view mode - it is always in edit state.

- Use for: creating new records and editing existing ones
- Does not support read-only or view mode
- Combine with lightning-input-field to control which fields appear

### **2\. lightning-record-view-form**

This component renders a read-only form, perfect for displaying record data on a detail page or in a panel. It always shows data in view mode and respects the field metadata for labels and formatting.

- Use for: displaying record data in a clean, read-only layout
- Supports custom layout using lightning-output-field
- Does not support editing or creating records

### **3\. lightning-record-form (The Swiss Army Knife)**

This is the most versatile of the three. A single component can handle create, edit, view, and read-only modes - all configurable via a mode attribute. It also supports layout types and multi-column layouts, making it the go-to choice for full-featured record pages.

**Key advantage:** You can specify a layout and allow admins to configure form fields declaratively through the page layout in Salesforce Setup. Alternatively, you can specify an ordered list of fields programmatically. Either way, lightning-record-form handles both edit and view of the same record in one component.

## **Feature Comparison Table**

Use this table to quickly identify which component fits your use case:

| **Feature**         | **lightning-record-form** | **lightning-record-view-form** | **lightning-record-edit-form** |
| ------------------- | ------------------------- | ------------------------------ | ------------------------------ |
| Create Records      | **✓**                     | -                              | **✓**                          |
| Edit Records        | **✓**                     | -                              | **✓**                          |
| View Records        | **✓**                     | **✓**                          | -                              |
| Read-Only Mode      | **✓**                     | **✓**                          | -                              |
| Layout Types        | **✓**                     | -                              | -                              |
| Multi Column Layout | **✓**                     | -                              | -                              |
| Custom Layout       | -                         | **✓**                          | **✓**                          |
| Custom Rendering    | -                         | **✓**                          | **✓**                          |

**Quick Selection Guide**

Need create + edit + view in one component? Use lightning-record-form. Need a custom read-only layout with your own rendering? Use lightning-record-view-form. Need a controlled edit/create form with validation? Use lightning-record-edit-form.

## **When Not to Use Base Lightning Components**

While Base Lightning Components are convenient, they have limitations. Consider moving to LDS wire adapters or Apex when you need:

- Complex logic before saving a record (multi-step wizards, conditional field visibility)
- Working with multiple related records simultaneously in one operation
- Non-standard UI that deviates significantly from Salesforce's layout engine
- Operations on records outside SOQL standard object support
- Fine-grained control over the exact HTTP request being made

## **Summary**

Base Lightning Components are the fastest way to build data-driven UIs in Salesforce LWC. Because they are built on Lightning Data Service, your components automatically benefit from client-side caching, UI API's rich metadata responses, and automatic cache invalidation - all without writing a single line of Apex or wire adapter code.

The three components - lightning-record-form, lightning-record-edit-form, and lightning-record-view-form - cover the vast majority of standard CRUD use cases. Start here, and only reach for LDS wire adapters or Apex when your requirements outgrow what these components offer.

**Key Takeaway**

Easiest to implement. Least code to write. Automatic caching, security, and validation. If your use case fits, Base Lightning Components are almost always the right first choice.

_- Salesforce LWC Developer Series • April 2024 -_