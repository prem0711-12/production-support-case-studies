<img width="1886" height="990" alt="Account_Check_DB(MYSQL)" src="https://github.com/user-attachments/assets/642e9554-7bbd-47d2-ad2c-24fb5a9fd2d1" /># Balance Sync Failure & Tote Betting Access Lost After Website Migration

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

## Step 2 — Multiple Account Discovery (CRM)

-  Further investigation using the player's SSN linked to the legacy account revealed that the player had created two additional accounts on the new website using different email addresses — separate from their original legacy account.
-  Account 1 — Created on Jun 11, 2026 at 23:06:25. Status: Approved. Registration Progress: Password step completed.
-  Account 2 — Created on Jun 16, 2026 at 00:07:25. Status: Approved. Registration Progress: Password step completed.
-  One of the accounts was found to be Deactivated by the player after creation.
-  This confirmed the player had 3 accounts in total — 1 legacy (not migrated) + 2 new website accounts (1 active, 1 deactivated).

Screenshot 1 
<img width="998" height="149" alt="Account_1" src="https://github.com/user-attachments/assets/87414caf-1a62-45ad-8ba1-43297e25033f" />

Screenshot 2 

<img width="1001" height="152" alt="Account_2" src="https://github.com/user-attachments/assets/db269886-e4a4-4591-b575-6f5846071cd0" />

Screenshot 3

<img width="991" height="82" alt="Account1_status" src="https://github.com/user-attachments/assets/dfaeda36-f429-43d1-a6d1-fc3193f3dc10" />

## Step 3 — Migration Status Confirmed via MySQL (DBeaver)

-  To double-confirm the migration status at the database level, a query was run against the ut_migration_data table using the player's tote account ID.
-  Result returned migration = 0 — confirming the player had not been migrated in the system database, consistent with what the CRM showed.

```sql
SELECT *
FROM ut_migration_data umd
WHERE umd.tote_acc_id = [tote_account_id];
```

-  1 row fetched. migration column = 0, migrated_at = NULL — no migration record exists for this player.

  <img width="1886" height="990" alt="Account_Check_DB(MYSQL)" src="https://github.com/user-attachments/assets/574ba8f8-b301-4289-8b8d-c076a250e3d3" />

