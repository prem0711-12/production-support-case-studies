# Deposited Funds But Balance Not Updated

## Customer Report
- A deposit of $40,000  was initiated.
- The charged amount was $40,067.
- The deposited funds were not reflected in the account balance.

<img width="1556" height="601" alt="ebb397c8-890b-436c-9148-79c555d21122" src="https://github.com/user-attachments/assets/fe0ec1e2-384f-4abf-a036-4c1e0824306c" />

## Systems Investigated
GCP Logs | MySQL | Cassandra | Internal Service
  
# Investigation Process

## Step 1 - GCP Logs

Objective:
-  To verify whether the deposit request was received, I checked GCP Logs after adjusting for the customer's timezone.
-  The logs showed a NullPointerException during the MercadoPay callback — paymentId was returning null.

Screenshot:

<img width="1168" height="259" alt="GCP_Logs_Investigation" src="https://github.com/user-attachments/assets/b4dd2b4f-e179-4563-960a-013feb690658" />

## Step 2 — MySQL Check

<img width="1886" height="1002" alt="MySQL" src="https://github.com/user-attachments/assets/05d34da2-4ac8-4892-806d-2f887da576f5" />

-  Query run against payment_info table to retrieve transaction history for the affected player. 
-  Results showed 2 deposits with txn_res_status = INITIATED (never completed), confirming the balance was not updated.

## Step 3 — Cassandra Database Check

<img width="1605" height="212" alt="Cassandra" src="https://github.com/user-attachments/assets/1adab6c8-00e1-41a3-bc3b-490eb9e1528d" />

Queried the Cassandra payments table using localdate and player ID to verify the transaction record.

Observation:
- 2 rows found with paymentstatus = PENDING
- txnid = null on both records
- This confirmed the payment gateway did not return a successful transaction ID, leaving both deposits stuck in PENDING state.
