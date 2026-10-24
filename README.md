# 🕵️‍♂️ Troubleshooting Hybrid Identity: The "Ghost User" UPN Conflict

## 📖 Overview
This lab documents a real-world Hybrid Identity troubleshooting scenario. While automating user onboarding via PowerShell and Entra Connect, I encountered a UPN (User Principal Name) sync conflict where the cloud rejected the custom domain and assigned a random `onmicrosoft.com` fallback address.

---

## 🛠️ The STAR Method Breakdown

### **S - Situation**
I automated the creation of 5 new IT department users using a PowerShell script reading from a CSV file. After successfully creating the users on-premises in Active Directory, I ran a Delta Sync (`Start-ADSyncSyncCycle -PolicyType Delta`) to push them to Microsoft Entra ID.

### **T - Task**
I needed to ensure all 5 users synced to the cloud with their correct custom UPN (`@ardypascual.site`) so they could be assigned licenses and access Microsoft 365 apps.

### **A - Action**
1. **Discovery:** I noticed one user, *Erwin Tulfo*, failed to get his custom domain. Instead, Entra ID assigned him a fallback UPN (`etulfo8303@evilcorpLAB.onmicrosoft.com`).
2. **Investigation:** I realized this was a "Soft-Match Conflict." A previous batch of test users had been deleted on-premises but were still sitting in the Entra ID recycle bin (soft-deleted). The cloud was holding onto the `etulfo` username to prevent accidental data loss.
3. **Resolution:**
   * Navigated to the **Microsoft Entra Admin Center** (not the M365 portal).
   * Went to **Users > Deleted Users** and permanently deleted the "ghost" accounts.
   * **Verification:** I checked the Entra ID Audit logs to confirm the "Hard delete user" actions were successful and initiated by my admin account (`ardy@ardypascual.site`).
   * Forced a Delta Sync again from the on-prem Domain Controller.

### **R - Result**
The permanent deletion cleared the reservation on the username. Entra Connect successfully re-evaluated the identity and updated Erwin's UPN to the correct **`etulfo@ardypascual.site`**. 

*This exercise reinforced the importance of managing the Entra ID Recycle Bin and utilizing Audit Logs when dealing with hybrid identity sync errors.*

---

## 📸 Proof of Work

**1. The PowerShell Script & Sync**
*The script successfully created the 5 IT users and the sync cycle reported "Success."*

<img width="1095" height="382" alt="bulkimport" src="https://github.com/user-attachments/assets/2091d7ef-2731-4492-a837-fc9a4f27efb0" />


**2. The Cloud Resolution**
*After permanently deleting the ghost accounts, the correct UPN `etulfo@ardypascual.site` was applied.*

<img width="1142" height="700" alt="fixUPNname" src="https://github.com/user-attachments/assets/f7e93b4c-6d03-47af-9f35-50c335139b6d" />


**3. Audit Log Proof (The Cleanup)**
*The Entra ID Audit Log confirms the "Hard delete user" actions were successful, which cleared the UPN reservation for the new sync.*

<img width="1881" height="377" alt="auditdeletedUsers" src="https://github.com/user-attachments/assets/6db25916-2e66-46ae-a150-ea93cc113a88" />


---

## 💡 Key Takeaways
*   **Hybrid Identity requires patience:** When you delete an on-prem user, Entra Connect marks the cloud user as "soft-deleted" for 30 days. 
*   **Always check the Recycle Bin:** If you delete and recreate a user with the same name, you must permanently delete the old cloud account first, or the sync will fail/assign random UPNs.
*   **Use the Right Portal:** The "Delete permanently" button is only available in the **Entra Admin Center**, not the standard M365 Admin Center.
*   **Audit Logs are your friend:** The Audit Logs clearly show the timestamp, the action ("Hard delete user"), the target object, and the admin who performed it, making it perfect for compliance and documentation.
