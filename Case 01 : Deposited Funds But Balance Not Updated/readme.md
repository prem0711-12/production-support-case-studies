# Deposited Funds But Balance Not Updated

## Customer Report
- A deposit of $40,000  was initiated.
- The charged amount was $40,067.
- The deposited funds were not reflected in the account balance.

<img width="1556" height="601" alt="ebb397c8-890b-436c-9148-79c555d21122" src="https://github.com/user-attachments/assets/fe0ec1e2-384f-4abf-a036-4c1e0824306c" />

## Systems Investigated
GCP Logs | MySQL | Cassandra | Internal Service
  
# Investigation Process

## Step 1 - GCP Alerting — How the Issue Was Already Flagged

-  To verify whether the deposit request was received, I checked GCP Logs after adjusting for the customer's timezone.
-  The logs showed a NullPointerException during the MercadoPay callback — paymentId was returning null.

Screenshot:

<img width="1168" height="259" alt="GCP_Logs_Investigation" src="https://github.com/user-attachments/assets/b4dd2b4f-e179-4563-960a-013feb690658" />

## Step 2 — MySQL Check


<img width="1886" height="1002" alt="MySQL" src="https://github.com/user-attachments/assets/05d34da2-4ac8-4892-806d-2f887da576f5" />



-  Query run against payment_info table to retrieve transaction history for the affected player. Results showed 2 deposits with txn_res_status = INITIATED (never completed), confirming the balance was not updated.

  

## Step 3 — Cassandra Database Check

<img width="1605" height="212" alt="Cassandra" src="https://github.com/user-attachments/assets/1adab6c8-00e1-41a3-bc3b-490eb9e1528d" />

Queried the Cassandra payments table using localdate and player ID to verify the transaction record.

Observation:
- 2 rows found with paymentstatus = PENDING
- txnid = null on both records
- This confirmed the payment gateway did not return a successful transaction ID, leaving both deposits stuck in PENDING state.

## Step 4 — Development Team Escalation & Root Cause

Since this was a live production issue affecting a customer's balance, I did not spend additional time on further investigation. After completing checks across GCP Logs, MySQL, and Cassandra, I summarized my findings and escalated to the development team via Slack.
As the incident occurred after working hours with no immediate response on the channel, I waited 10 minutes before directly calling the on-call developer to ensure it was picked up without further delay.

Developer Finding:
-  After reviewing the logs and payment flow, the developer confirmed the root cause — the payment gateway had sent a callback to our system with paymentId returning as null. Since our application requires a valid paymentId to proceed with processing, the transaction was rejected internally and remained stuck in PENDING state in both MySQL and Cassandra.

Resolution:

The payment gateway team was notified about the null paymentId being returned in their callback response. They acknowledged the issue and applied a fix on their end.

Additional Context — GCP Alerting:

-  During this investigation it was also noted that our GCP alerting system had already flagged this error before the customer reached out. Our applications running on GCP are connected to Slack via alerting policies — when specific critical keywords such as error appear in application logs, an automatic notification is sent to the designated Slack channel along with the application name.
-  This means the issue was visible in our monitoring before it was reported — something worth noting for future incident response timing.
