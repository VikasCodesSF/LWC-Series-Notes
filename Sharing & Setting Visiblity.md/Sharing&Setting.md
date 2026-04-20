# Salesforce Sharing Model – Key Notes

> **Audience:** Advanced System Administrators & Architects  
> **Prerequisite Knowledge:** Salesforce security and sharing model basics

---

## 1. Types of Data Access

- Record-level security controls **which records** a user can see (not just which objects).
- **Record ownership = Full access** — the highest access level on a record.
- Users higher in the **role/territory hierarchy** inherit access to their subordinates' records (standard objects only).

### License Considerations

| License Type | Sharing Model | Notes |
|---|---|---|
| Customer Community / High-Volume | **No standard sharing model** | Uses foreign key match on Account/Contact; needs Sharing Sets or Share Groups |
| Chatter Free / Chatter External | **No sharing** | No CRM record access, no roles |
| Full Salesforce License | Standard sharing model | Covered by all components below | 

---

## 2. Core Sharing Components

### 2.1 Profiles & Permission Sets
- Control **object-level** and **field-level** security.
- **View All / Modify All** (per object) override sharing rules entirely.
- ✅ Best Practice: Use **permission sets and permission set groups** instead of profiles to avoid managing hundreds of profiles.

### 2.2 Record Ownership & Queues
- Every record must have **one owner** (user or queue).
- Owner access is bound by the owner's profile permissions.
- Queues help distribute and assign records to teams.
- ⚠️ Best Practice: If a single user owns **>10,000 records**:
  - Do not assign that user a role in the hierarchy.
  - If a role is required, place it at the **top of its own hierarchy branch**.

### 2.3 Organization-Wide Defaults (OWD)
- Sets the **most restrictive baseline** for record access across the org.
- Other tools selectively open access above this baseline.
- Changing OWD requires **sharing recalculation** — can be time-consuming at large volumes.
- Custom objects only: **Grant Access Using Hierarchies** setting (unchecked = managers don't inherit subordinate access).

### 2.4 Role Hierarchy
- Ensures managers always have at least the **same access as their subordinates**.
- Does not need to mirror the HR org chart — model by **data access level**, not reporting structure.
- Limits:
  - Orgs created **Spring '21+**: up to **5,000 roles**
  - Older orgs: up to **500 roles** (expandable via support)
  - Best practice: keep internal roles ≤ **25,000**, external ≤ **100,000**
  - Best practice: keep hierarchy depth ≤ **10 levels**
- Peers in the same role do **not** automatically see each other's data.
- **Overlays** (cross-branch users) need sharing rules, teams, or territory management.

**Role Hierarchy Use Cases:**
- Management access & reporting roll-up
- Segregation between business units
- Private data within a role (leaf-node privacy)

### 2.5 Public Groups
- Collections of users, roles, territories, and other groups.
- Group membership alone does **not** grant data access — must be paired with sharing rules.
- Nesting supported (Group A inside Group B) but keep to ≤ **5 levels**.
- Best practice: keep total groups ≤ **100,000**.
- Can use **Grant Access Using Hierarchies = OFF** to make data accessible only to group members (not their managers).

### 2.6 Owner-Based Sharing Rules
- Grant access based on **record ownership**.
- Provide exceptions to OWD and role hierarchy.
- Limit: **300 sharing rules per object** (total).
- Use for: cross-branch role access, peer access, department-to-department access.
- ⚠️ Does NOT apply to private contacts (contacts not linked to an account).

### 2.7 Criteria-Based Sharing Rules
- Grant access based on **field values** on the record (not ownership).
- Limit: **50 criteria-based + guest user sharing rules per object**.
- Use when: access should open based on a field condition (e.g., Status = "Approved").

### 2.8 Guest User Sharing Rules
- Special type of criteria-based sharing rule for **unauthenticated users**.
- ⚠️ Warning: Grants **immediate, unlimited access** to all matching records to anyone — use with extreme caution.
- Limit: counted within the 50 criteria-based rule limit per object.

### 2.9 Manual Sharing
- Record owners (or admins) manually grant read or read/write access on individual records.
- Not automated — provides flexibility for one-off access.
- ⚠️ Manual shares are **removed** when the record owner changes.
- Share records with `row cause = Manual Share` can be managed via the Share button, even if created via code.

### 2.10 Teams (Account, Opportunity, Case)
- A group of users collaborating on a single record; supports mixed access levels (read-only vs. read/write).
- Only **one team per record**.
- Creating a team member creates **two records**: a team record + a share record (manage both if using code).
- If multiple teams are needed, consider territory management or programmatic sharing.

### 2.11 Territory Hierarchy (Enterprise Territory Management)
- Separate hierarchy from the role hierarchy, used to model **sales territories**.
- If using both territory-based AND role-based forecasting: maintain **both hierarchies**.
- Best practice: use role hierarchy for HR reporting, territory hierarchy for sales structure.
- ⚠️ Do NOT make role and territory hierarchies identical — causes unnecessary sharing activity.

### 2.12 Apex Managed Sharing (Programmatic Sharing)
- Use code to build **dynamic, sophisticated sharing** when no declarative method fits.
- Can use standard `manual share` row cause (subject to same deletion rules as manual shares).
- Can create **custom Apex sharing reasons** on custom objects for cleaner code.
- Use cases: external system of truth for access, large data volume performance issues, team-like functionality on custom objects.

### 2.13 Restriction Rules
- Work **opposite** to sharing rules — they **filter down** already-granted access.
- Available for: custom objects, external objects, contracts, events, tasks, time sheets, time sheet entries.
- Limits:
  - Enterprise/Developer: **2 active rules per object**
  - Performance/Unlimited: **5 active rules per object**
- Applied to: list views, lookups, related lists, reports, search, SOQL, SOSL.
- Use for: sensitive/confidential records, truly private contracts/tasks/events.

### 2.14 Implicit Sharing
- **Cannot be configured** — automatic and built into the platform.
- **Parent implicit sharing**: If a user can see an Opportunity, Case, or Contact → they automatically see the associated **Account** (read-only).
- **Child implicit sharing**: Account owner automatically gets access to associated Contacts, Opportunities, and Cases based on role settings (View / Edit / No Access).
- ⚠️ Does **not** apply to custom objects.

---

## 3. Customer Scenarios Summary

### Scenario A: Team Assignment Managed by External System

| Requirement | Solution |
|---|---|
| Manager needs access to another region | Owner-based sharing rule |
| Country ops need access to all country sales data | Owner-based sharing rule |
| "Core 4" team per account from external system | Account Teams |
| Managers need same access as subordinates | Role hierarchy |
| Teams must not be user-modifiable | Account Teams + remove team page layout |
| "Buddy" coverage for absent reps | Account Teams (buddy as a team role) |
| Custom deal needs non-sales members | Opportunity Teams (manual or trigger-based) |

### Scenario B: Out-of-Box Territory Management

| Requirement | Solution |
|---|---|
| Two business units need access to same accounts with separate hierarchy | Territory Management |
| Business developers assigned to specific accounts across teams | Sub-territories or separate territory branches |
| One-off account access for support roles | Account Teams |
| Credit dept needs all accounts for a business unit | Owner-based sharing rule OR territory modeling |
| Managers need subordinate access | Role hierarchy |

---

## 4. Key Considerations

### Realignment & Reassignment
- **Membership changes**: can happen frequently (daily/hourly) — lower cost.
- **Structural changes (realignments)**: expensive — should happen no more than **quarterly**.
- All bulk/mass changes should be **planned, tested, and coordinated** (preferably in sandbox first).

### Large Data Volumes
- Performance becomes a concern at scale — always test in sandbox.
- If **>2 million accounts** + teams or territory management: pay close attention to performance.

### Defer Sharing Calculations
- A Salesforce Support-enabled feature that suspends automatic recalculations during bulk changes.
- Allows all changes to be made first, then recalculate **once** — more efficient for large bulk operations.

### Data & Ownership Skews
- **Data skew**: few parent records with many children (e.g., one Account → 10,000+ Contacts). Ratio to watch: **1:10,000**.
- **Ownership skew**: one user/role/group owns many records. Same threshold: **1:10,000**.
- Both can cause long-running transactions and performance degradation during changes.

### Account Hierarchies ≠ Data Access
- A parent/child relationship between accounts does **not** grant access to child account records.
- Only role and territory hierarchies provide cascading data access.

---

## 5. Troubleshooting — Why Can't a User See a Record?

1. ✅ Verify the user has **object-level permission** (profile/permission set).
2. 🔍 Identify the **user's role** and the **record owner's role**.
3. 🔍 Confirm the two roles are in **different branches** of the hierarchy.
4. 🔍 Review **sharing rules** for the object — check public groups the user may be missing from.
5. 🔍 If using **Teams** — should this user be on the team? How was the miss introduced?
6. 🔍 If using **manual sharing** — was access lost due to owner change? Was share manually removed?
7. 🔍 If using **Enterprise Territory Management** — is the user in the correct territory? Is the record assigned to that territory?
8. 🔍 If using **programmatic sharing** — review the code logic to see why the share record wasn't created.

---

## 6. Quick Reference: Sharing Rule Limits

| Rule Type | Limit |
|---|---|
| Total sharing rules per object | 300 |
| Criteria-based sharing rules per object | 50 (shared with Guest User rules) |
| Guest user sharing rules per object | 50 (shared with Criteria-based rules) |
| Restriction rules (Enterprise/Developer) | 2 per object |
| Restriction rules (Performance/Unlimited) | 5 per object |
| Public group nesting depth | 5 levels max |
| Total public groups (org-wide) | 100,000 |
| Role hierarchy depth | 10 levels max (best practice) |
| Roles (orgs created Spring '21+) | 5,000 (best practice: ≤25,000 internal, ≤100,000 external) |