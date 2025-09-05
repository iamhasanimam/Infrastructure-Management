# 🏛 FSMO Roles in Active Directory

**FSMO** stands for **Flexible Single Master Operations**.  
These are **special roles** in Active Directory (AD) where only **one Domain Controller (DC)** at a time can perform specific tasks to prevent conflicts.  

There are **five FSMO roles**:

- **2 Forest-wide roles** (only **one per forest**).  
- **3 Domain-wide roles** (one **per domain**).

---

## 🌍 Forest-wide FSMO Roles

### 1. **Schema Master**
- **Full Form:** *Schema Flexible Single Master Operation*  
- **What it does:**  
  - Controls updates and modifications to the **Active Directory schema** (the blueprint that defines all object types and attributes).  
- **Real Example:**  
  - Installing **Exchange Server** requires extending the schema with new attributes like `mailNickname`.  
  - Only the **Schema Master** can make these changes.  
- **If it’s down:**  
  - Normal AD operations continue.  
  - Schema updates (e.g., installing new apps like Exchange or Skype for Business) cannot happen.

---

### 2. **Domain Naming Master**
- **Full Form:** *Domain Naming Flexible Single Master Operation*  
- **What it does:**  
  - Ensures that **domain names are unique** inside the forest.  
- **Real Example:**  
  - Adding a new domain `sales.corp.local`.  
  - The **Domain Naming Master** verifies no duplicate domain exists in the forest.  
- **If it’s down:**  
  - You cannot **create or delete domains** in the forest.  
  - Existing domains continue working fine.

---

## 🏠 Domain-wide FSMO Roles

### 3. **RID Master**
- **Full Form:** *Relative Identifier Flexible Single Master Operation*  
- **What it does:**  
  - Assigns **RID pools** to Domain Controllers.  
  - RIDs combine with Domain IDs to form **unique SIDs** (Security Identifiers) for users, groups, and computers.  
- **Real Example:**  
  - You create 100 new users.  
  - Your DC uses its RID pool from the **RID Master** to assign unique SIDs.  
- **If it’s down:**  
  - New objects can still be created **until RID pools run out**.  
  - Once pools are empty, no new users/groups can be created.

---

### 4. **PDC Emulator**
- **Full Form:** *Primary Domain Controller Emulator Flexible Single Master Operation*  
- **What it does (most critical):**  
  - **Password updates:** Verifies password changes instantly.  
  - **Time synchronization:** Acts as the domain’s time server (essential for Kerberos authentication).  
  - **Lockouts:** Processes account lockouts.  
  - **Legacy support:** Acts as NT4 PDC for backward compatibility.  
- **Real Example:**  
  - A user changes their password on DC02 but logs in via DC01.  
  - DC01 queries the **PDC Emulator** to confirm the new password immediately.  
- **If it’s down:**  
  - Password changes may take longer to replicate.  
  - Time drift issues may cause Kerberos failures.  
  - Lockout handling may be delayed.  

---

### 5. **Infrastructure Master**
- **Full Form:** *Infrastructure Flexible Single Master Operation*  
- **What it does:**  
  - Maintains references to objects in **other domains**.  
  - Updates group membership references when users move domains.  
- **Real Example:**  
  - User **Alice** from `HR.corp.local` is added to a group in `Sales.corp.local`.  
  - The **Infrastructure Master** ensures Alice’s SID and name remain updated in Sales domain.  
- **If it’s down:**  
  - Cross-domain group membership may not update properly.  
  - Not an issue if **all DCs are Global Catalog servers**.

---

## 📌 FSMO Roles Quick Reference

| Scope      | FSMO Role            | Full Form | Responsibility | Real Example |
|------------|----------------------|-----------|----------------|--------------|
| **Forest** | Schema Master        | Schema Flexible Single Master Operation | Controls schema updates | Extending schema for Exchange |
| **Forest** | Domain Naming Master | Domain Naming Flexible Single Master Operation | Maintains unique domain names | Adding `sales.corp.local` |
| **Domain** | RID Master           | Relative Identifier Flexible Single Master Operation | Issues RID pools for unique SIDs | Creating 100 new users |
| **Domain** | PDC Emulator         | Primary Domain Controller Emulator Flexible Single Master Operation | Passwords, time, lockouts, legacy | User password reset + time sync |
| **Domain** | Infrastructure Master| Infrastructure Flexible Single Master Operation | Updates cross-domain references | HR user in Sales group |

---

## 🏗 FSMO Architecture Diagram (Textual)

```
                  Forest (corp.local)
                  ───────────────────
                     ⬇ FSMO Roles
               ┌───────────────────────┐
               │   Schema Master        │
               │   Domain Naming Master │
               └───────────────────────┘
                           │
           ┌──────────────────────────────────┐
           │            Domain                 │
           │          (corp.local)             │
           │                                    │
           │   ┌──────────────────────────┐     │
           │   │ RID Master               │     │
           │   │ PDC Emulator             │     │
           │   │ Infrastructure Master    │     │
           │   └──────────────────────────┘     │
           └──────────────────────────────────┘
```

---

⚡ **Key Takeaway:**  
- Forest roles = **once per forest** (Schema, Domain Naming).  
- Domain roles = **once per domain** (RID, PDC, Infrastructure).  
- **PDC Emulator** is the most critical for day-to-day operations.  
