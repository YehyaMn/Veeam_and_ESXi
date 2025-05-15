# Production Server Storage Upgrade and Failover – Lenovo Servers with ESXi & Veeam



## Environment
- **Servers**: Lenovo Rack Servers (Production + Replica)
- **Hypervisor**: VMware ESXi
- **Backup & Replication**: Veeam Backup & Replication
- **Management Tools**:
  - **Lenovo XClarity** (Replica Server)
  - **System Configuration Utility via Boot Menu** (Production Server)

---

## Objective

Redesign and upgrade the storage layout to:
- Separate **ESXi OS** and **VM Datastore**
- Improve reliability and recovery speed
- Test and perform **failover/failback** using Veeam

---

##  Storage Configuration

###  Previous Setup
- **RAID 5** (3x SSD 1.75TB):  
  - Shared both **ESXi OS** and **VMs**

### New Setup (on both servers)
- **RAID 1** (2x SSD):  
  - ESXi hypervisor
- **RAID 5** (3x SSD 1.75TB):  
  - Dedicated VM Datastore (approx. 3.5TB usable)

---

##  Technical Steps

### Step 1: RAID Reconfiguration

#### Replica Server (via Lenovo XClarity):
1. Add 5 SSDs (2 for RAID 1, 3 for RAID 5).
2. Log into Lenovo XClarity Controller.
3. Delete existing virtual drive.
4. Create:
   - RAID 1 (2 SSDs) → For ESXi
   - RAID 5 (3 SSDs) → For VM Datastore
5. Install fresh ESXi and configure network.

#### Production Server (via Boot Utility):
1. Boot into Lenovo System Configuration utility.
2. Delete old RAID 5 volume.
3. Create RAID 1 + RAID 5 as above.
4. Reinstall ESXi and match config to replica server.

---
![WhatsApp Image 2025-05-15 at 8 38 14 PM](https://github.com/user-attachments/assets/26ba9187-112c-4c35-a708-d15e4eb7943e)

###  Step 2: Veeam Replication Setup

1. Open **Veeam Backup & Replication Console**.
2. Go to **Replication Jobs > Create Replication Job**.
3. Select target VMs (DC, DB, etc.).
4. Set **Replica Target**:
   - Host: Replica ESXi
   - Datastore: RAID 5 volume
5. Enable **Application-aware processing** (for proper DB/DC shutdown).
6. Set replication schedule or run manually.
7. Wait for job to complete (should be up-to-date before failover).

![WhatsApp Image 2025-05-15 at 8 37 37 PM](https://github.com/user-attachments/assets/5049c985-cb35-4e9c-b298-4ce930d57e9f)

---

###  Step 3: Perform Planned Failover (Graceful)

1. Navigate to **Replicas > Ready**.
2. Right-click the replicated VM → **Failover Now**.
3. Select the **Restore Point** (the latest).
4. Enable checkbox for **"Planned Failover"**:
   - This gracefully powers off source VM and avoids data loss.
5. Confirm → Wait for replica VM to boot.
6. Confirm services (e.g. DC, DB) are running.

 **Downtime in this case: ~5 minutes**

---
![WhatsApp Image 2025-05-15 at 8 36 43 PM](https://github.com/user-attachments/assets/2f1d489c-f3f0-4ed4-95be-f82b1015099a)

###  Step 4: Rebuild Production Server

1. After verifying the replica is working:
   - Rebuild RAID and reinstall ESXi on production server.
   - Match ESXi version, IPs, and vSwitch setup.
2. Register host in Veeam again if needed.

---

###  Step 5: Commit Failback to Production

1. In Veeam console, go to the **Replica VM**.
2. Click **Failback to Production**.
3. Select:
   - Source host: New production ESXi
   - Datastore: New RAID 5
4. Choose **Restore to Original Location** (or map if paths changed).
5. Optionally enable:
   - **Power off Replica VM**
   - **Power on Recovered VM**
6. Monitor the data sync and switchover process.
7. Once stable, **Commit Failback**.

 You can also use the **"Undo Failover"** if needed before commit, to return to the replica state.

---

##  Outcome

- Storage layout is now more fault-tolerant and modular.
- VMs were successfully failed over with <5 min downtime.
- Production and Replica are aligned, improving DR posture.

---

##  Tags

`VMware` `ESXi` `Veeam` `RAID` `Lenovo` `XClarity` `Failover` `Replication` `DisasterRecovery` `SysAdmin` `Infrastructure`
