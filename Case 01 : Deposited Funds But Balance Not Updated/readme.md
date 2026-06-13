# Deposited Funds But Balance Not Updated

## Customer Report

Customer reported that:

- A deposit of $40,000  was initiated.
- The charged amount was $40,067.
- The deposited funds were not reflected in the account balance.

<img width="1556" height="601" alt="ebb397c8-890b-436c-9148-79c555d21122" src="https://github.com/user-attachments/assets/fe0ec1e2-384f-4abf-a036-4c1e0824306c" />

## Systems Involved

- Google Cloud Platform (GCP) Logs
- MySQL
- Cassandra
- Internal Service
  
# Investigation Process

## Step 1 - Review Application Logs

Objective:
- Verify whether the deposit request was received and processed successfully.
- As the customer was in a different time zone, I first adjusted the time accordingly before searching the logs.
- After identifying the correct time window, I checked the GCP logs for the transaction.

Tool Used:
- GCP Logs

Observation:
- An error was identified during the deposit flow.

Screenshot:
<img width="1168" height="259" alt="GCP_Logs_Investigation" src="https://github.com/user-attachments/assets/b4dd2b4f-e179-4563-960a-013feb690658" />

