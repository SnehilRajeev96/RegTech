# RegTech
**1. CoordinationAgent**

**Purpose**

•	Central orchestrator that routes incoming HTTP requests and ComplianceMessage objects between agents.

**Functionality**

•	Exposes an HTTP POST /compliance endpoint.

•	Parses type and data from incoming requests and forwards to the appropriate agent (RegWatcherAgent, AMLAlertAgent, or KYCValidatorAgent).

•	Listens for ComplianceMessage events and logs each message source and payload.

**Testing**
1.	Deploy & start the agent.

2.	Send an HTTP POST to /compliance with JSON { "type":"transaction", "data":{...} }.

3.	Verify AMLAlertAgent receives the payload.

4.	Send a ComplianceMessage manually via UI and confirm logs show receipt.

**2. RegWatcherAgent**

**Purpose**

•	Periodically fetch and summarize new regulatory updates from UAE regulators (CBUAE, DFSA, ADGM).

**Functionality**

•	On startup (and at scheduled intervals), retrieves the latest items from configured RSS feeds.

•	Extracts top headlines and links into a structured list.

•	Sends a ComplianceMessage with the updates payload to CoordinationAgent.

**Testing**

1.	Deploy & start the agent.

2.	Check logs to see successful fetch of RSS items.

3.	Confirm CoordinationAgent logs a ComplianceMessage from RegWatcherAgent with the updates list.


**3. AMLAlertAgent**

**Purpose**

•	Monitor incoming financial transactions and flag potential AML red flags.

**Functionality**

•	Listens for Transaction messages with tx_id and details.

•	Applies rule-based checks (e.g., amount threshold).

•	If criteria met, sends a ComplianceMessage with an alert payload to CoordinationAgent.

**Testing**

1.	Deploy & start the agent.

2.	Send a Transaction via UI: { "tx_id":"T100", "details":{"amount":200000} }.

3.	Verify CoordinationAgent logs a ComplianceMessage from AMLAlertAgent containing the alert details.

**4. KYCValidatorAgent**

**Purpose**

•	Validate KYC requests against internal sanction and PEP lists.

**Functionality**

•	Listens for KYCRequest messages with user_id and doc_data.

•	Checks the user_id against hardcoded or external sanction/PEP lists.

•	Sends a ComplianceMessage with kyc_result indicating passed or listing issues to CoordinationAgent.

**Testing**

1.	Deploy & start the agent.

2.	Send a KYCRequest via UI: { "user_id":"EvilCorp", "doc_data":{...} }.

3.	Confirm CoordinationAgent logs a ComplianceMessage showing kyc_result with failure issues.

**5. RiskScorerAgent**

**Purpose**

•	Aggregate AML alerts and KYC results into a unified risk score for each entity.

**Functionality**

•	Listens for ComplianceMessage events with payloads containing alert or kyc_result.

•	Starts from a baseline score (0.1); adds 0.5 for alerts, 0.3 for KYC failures; caps at 1.0.

•	Sends a new ComplianceMessage with risk_score and details to CoordinationAgent.

**Testing**

1.	Deploy & start the agent.

2.	Send a ComplianceMessage with alert payload via UI.

3.	Check CoordinationAgent logs a new message from RiskScorerAgent with the calculated risk_score.

**6. ReportGenAgent**

**Purpose**

•	Compile a running list of all ComplianceMessage events into a consolidated report.

**Functionality**

•	Listens for ComplianceMessage events.

•	Appends each entry with timestamp to an in-memory list.

•	Creates a Report object (report_id, timestamp, entries) and sends it to NotifierAgent.

**Testing**

1.	Deploy & start the agent.

2.	Send multiple ComplianceMessage events.

3.	Verify NotifierAgent logs receipt of a Report containing all entries in the session.

**7. NotifierAgent**

**Purpose**

•	Dispatch or log finalized reports for downstream consumption (e.g., dashboard, email).

**Functionality**

•	Listens for Report messages with report_id, timestamp, and entries.

•	Logs a summary message about report receipt.

•	Optionally, forwards the report JSON to an external webhook or API endpoint.

**Testing**

1.	Deploy & start the agent.

2.	Send a Report via UI: { "report_id":"...", "timestamp":"...", "entries":[...] }.

3.	Confirm log entry: Notifier received report ... with n entries and/or successful webhook POST.

