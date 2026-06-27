# Balance Sync Failure & Tote Betting Access Lost After Website Migration

## Customer Report

-  Customer reported their account balance was not reflecting correctly on the website
-  Customer was unable to place bets through the tote phone betting system

<img width="1460" height="220" alt="Redmile_Customer_Query_Slack" src="https://github.com/user-attachments/assets/7fc2319b-b8fa-47e8-912e-e485a34fab2d" />


## Systems Investigated

CRM | Firebase | MySQL | Tote API (Postman) | Slack

# Investigation Process

## Step 1 — Legacy Player Check (CRM)

-  To begin the investigation, the customer's tote account ID was searched in our internal CRM under the Legacy Players section.
-  The CRM confirmed the player had a legacy account with a balance of $197.23 at the time of import from the legacy system.
-  The migration status showed "Not Migrated" — confirming the player had never completed the migration process to the new website.

<img width="1386" height="458" alt="Legacy_Player_check" src="https://github.com/user-attachments/assets/387826e1-3b37-4b8a-bfd9-ef1ba974a9ec" />

## Step 2 — Client Update & Account Confusion Identified (Slack)

<img width="549" height="639" alt="Further_discussion_with_cilent" src="https://github.com/user-attachments/assets/7781fa7c-ae0e-4203-9f08-60f3177f0186" />


-  The client informed us via Slack that the customer had attempted to log in but was shown an account deactivated message. The customer had since logged in using new credentials, had funded the new account, but remained unable to place any bets.
-  The client also confirmed that the customer had other accounts on the system.
-  After reviewing all findings from the investigation, the root cause of the betting access issue was identified and communicated back to the client — this was      not a system error but a result of how the player had set up their accounts:
  -  The customer had visited the website and created a new account using a new email address and password — these new website credentials are not linked to the        tote system and cannot be used for phone betting.
  -  The customer's original tote account had not been migrated to the new website. Until the migration is completed using the original tote credentials, phone         betting through the tote system remains inaccessible.

-  This explanation was provided after completing the full account investigation detailed in the steps below.



## Step 3 — Multiple Account Discovery (CRM)

-  Further investigation using the player's SSN linked to the legacy account revealed that the player had created two additional accounts on the new website using different email addresses — separate from their original legacy account.
-  Account 1 — Created on Jun 11, 2026 at 23:06:25. Status: Approved. Registration Progress: Password step completed.
-  Account 2 — Created on Jun 16, 2026 at 00:07:25. Status: Approved. Registration Progress: Password step completed.
-  One of the accounts was found to be Deactivated by the player after creation.
-  This confirmed the player had 3 accounts in total — 1 legacy (not migrated) + 2 new website accounts (1 active, 1 deactivated).

**Screenshot 1 — Account 1**
 
<img width="998" height="149" alt="Account_1" src="https://github.com/user-attachments/assets/87414caf-1a62-45ad-8ba1-43297e25033f" />

**Screenshot 2 — Account 2**

<img width="1001" height="152" alt="Account_2" src="https://github.com/user-attachments/assets/db269886-e4a4-4591-b575-6f5846071cd0" />

**Screenshot 3 — Account 1: Status - Deactivated**

<img width="991" height="82" alt="Account1_status" src="https://github.com/user-attachments/assets/dfaeda36-f429-43d1-a6d1-fc3193f3dc10" />



## Step 4 — Migration Status Confirmed via MySQL (DBeaver)

-  To double-confirm the migration status at the database level, a query was run against the ut_migration_data table using the player's tote account ID.
-  Result returned migration = 0 — confirming the player had not been migrated in the system database, consistent with what the CRM showed.

```sql
SELECT *
FROM ut_migration_data umd
WHERE umd.tote_acc_id = [tote_account_id];
```

-  1 row fetched. migration column = 0, migrated_at = NULL — no migration record exists for this player.

  <img width="1886" height="990" alt="Account_Check_DB(MYSQL)" src="https://github.com/user-attachments/assets/574ba8f8-b301-4289-8b8d-c076a250e3d3" />
  
## Step 5 — Balance Mismatch Identified (Firebase vs CRM vs Tote API)

-  While investigating the active account, the wallet balance displayed on the website did not match the actual balance held in the tote system.
-  Checked the player's wallet directly in Firebase Firestore under the CUSTOMERS → WALLET path — Firebase showed drawAmt: 211.14.
-  Checked the same player's wallet in the CRM admin panel under the Wallets tab — balance showed $213.24, confirmed as Primary, Active, FIAT wallet.
-  To verify the actual balance held on the tote side, a POST request was made via Postman using the fetch balance endpoint under the account collection — the API response returned ```<Balance>213.24</Balance>```.
-  This confirmed a sync discrepancy — Firebase was holding a stale value of $211.14 while the actual tote balance was $213.24.

**Screenshot 1 — Tote API Balance Verification (Postman)**

<img width="1498" height="956" alt="Verfied_With_Actual_Tote_V1" src="https://github.com/user-attachments/assets/d2a2a608-4206-42f3-8cc9-b51aaf3c37ba" />

**Screenshot 2 — Wallet Balance in CRM Admin**

<img width="1313" height="784" alt="Balance_Verification_Tenant_admin_V1" src="https://github.com/user-attachments/assets/675c358d-dba5-4f81-a967-4a40fc3ca402" />

**Screenshot 3 — Firebase Firestore Wallet Value**

<img width="1858" height="970" alt="Balance_verification_on_Firebase" src="https://github.com/user-attachments/assets/34d35999-6ff6-4369-88c6-795a7a506c46" />


## Step 6 — Balance Sync & Resolution

-  The wallet balance displayed on the website ($211.14 in Firebase) did not match the actual balance confirmed via the CRM admin panel and Tote API ($213.24).
-  Since the wallet is managed externally and cannot be modified directly from the admin panel, the "Sync Wallets" button available in the CRM was used to manually trigger a balance sync.
-  After the sync, the website balance updated to reflect the correct amount of $213.24, resolving the display mismatch without any developer escalation.

#  Root Cause Summary:

##  Issue A 
Balance Mismatch: Firebase was holding a stale wallet value that had not synced with the actual tote balance. Resolved via manual wallet sync triggered from the CRM admin panel.

##  Issue B 

Phone Betting Access: The customer's legacy tote account had never been migrated to the new website. Instead, the customer created two new accounts using an email address that was previously associated with their original tote account, which remained unmigrated. As a result, the new accounts were not linked to the original tote credentials, making phone betting through the tote system inaccessible. The client was advised that the customer must first migrate their legacy account using their original tote credentials — once migration is complete, their original tote account ID will be active on the new platform and phone betting access will be restored.

  

