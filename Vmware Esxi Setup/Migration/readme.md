### Migration of server roles such as DNS DHCP AND ADDS on a new server

# Active Directory Lab Documentation

This file summarizes everything we have done so far in your ESXi-based Active Directory lab: **theory + practical steps + why we did it**.

---

## 1. Installing Domain Controllers (DC01, DC02)
- **DC01** was the first Domain Controller (root DC) for the forest `infra.local`.
- **DC02** was promoted as an additional Domain Controller.

**Why?**  
- AD requires redundancy. Adding DC02 allows us to later decommission DC01 safely.  
- Multiple DCs also ensure replication and fault tolerance.

---

## 2. Checking Schema Version
We ran in PowerShell:
```powershell
Get-ADObject (Get-ADRootDSE).schemaNamingContext -Property objectVersion
```

- **Result = 88**
- Meaning: The AD schema is at Windows Server 2019/2022 level.

**Why?**  
- Schema defines the "dictionary" of all AD objects.  
- Each Windows Server version may introduce new attributes.  
- Knowing the schema version ensures compatibility before adding new DCs.

---

## 3. ADPREP (Forestprep & Domainprep)
We mounted the **Windows Server 2022 ISO** and ran from CMD (Admin):
```cmd
adprep.exe /forestprep
adprep.exe /domainprep
```

- `forestprep` → once per forest (on Schema Master).  
- `domainprep` → once per domain.  

**Why?**  
- Updates schema and domain permissions to support new DCs.  
- Without this, a 2022 DC promotion could fail in older forests.  
- In our case, schema was already at 88, so no change was needed.

---

## 4. Understanding Replication
- Used tools like:
  ```powershell
  repadmin /replsummary
  repadmin /showrepl
  dcdiag /v
  ```

**Why?**  
- Replication ensures every DC has the same AD database.  
- Before FSMO transfer or DC decommission, replication must be healthy.

---

## 5. FSMO Roles
There are 5 FSMO roles:
- **Forest-wide**  
  - Schema Master  
  - Domain Naming Master  
- **Domain-wide**  
  - PDC Emulator  
  - RID Master  
  - Infrastructure Master  

We checked FSMO roles:
```powershell
netdom query fsmo
```

---

## 6. Transferring FSMO Roles
We moved FSMO roles from **DC01 → DC02**:
```powershell
Move-ADDirectoryServerOperationMasterRole -Identity DC02 -OperationMasterRole 0,1,2,3,4
```
(or one by one).

**Why?**  
- DC01 is planned for decommission.  
- FSMO roles must be owned by a live DC.  
- DC02 now controls AD operations.

Verification:
```powershell
netdom query fsmo
```
Output confirmed all FSMO roles now belong to **DC02**.

---

## 7. DHCP & DNS (Next Step)
- DNS role should be installed on DC02 (ensures name resolution continues after DC01 is gone).  
- DHCP database should be migrated using:
  ```powershell
  netsh dhcp server export C:\dhcp.txt all
  netsh dhcp server import C:\dhcp.txt all
  ```

---

## 8. Demoting DC01 (Planned)
- After FSMO roles, DNS, and DHCP are moved → DC01 can be demoted.  
- Done via **Server Manager → Remove Roles → uncheck AD DS**.  

**Why?**  
- To simulate DC lifecycle management (retire old DCs, move services).  
- Ensures DC02 is now the **sole DC** running AD DS, DNS, DHCP.

---

# ✅ Key Learnings

1. **Schema** = Blueprint of AD objects.  
   - Checked version (88 = 2019/2022).  
   - Used `adprep` for upgrades.

2. **Replication** = Synchronization between DCs.  
   - Verified with `repadmin` and `dcdiag`.  

3. **FSMO Roles** = Special AD tasks.  
   - Moved from DC01 to DC02 before decommission.  

4. **DNS & DHCP** = Core services.  
   - Must be migrated to the new DC.  

5. **Decommissioning DC01** = Safe retirement after role transfer + replication check.

---

# 🔮 Next Steps

1. Confirm DC02 is **Global Catalog** and **DNS server**.  
2. Migrate DHCP from DC01 → DC02.  
3. Verify replication one last time.  
4. Demote DC01 safely.  
5. Document final state: DC02 = sole DC for `infra.local`.


