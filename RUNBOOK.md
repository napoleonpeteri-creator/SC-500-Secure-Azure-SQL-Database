RUNBOOK: SC-500 LAB 2B - SECURE AZURE SQL DATABASE
============================================================

PURPOSE
Secure an Azure SQL Database by implementing:
1. Microsoft Entra ID administration
2. Network isolation with Private Endpoint
3. Public access restriction
4. SQL auditing to Log Analytics
5. Audit-event verification with KQL
6. Microsoft Defender for SQL

LAB RESOURCES
SQL Server:        sc500-lab2b-sql
Database:          ai-workload-db
Log Analytics:     sc500-lab2b-log
VNet:              sc500-lab2b-vnet
Entra group:       sc500-sql-admins
Test table:        AiModelMetadata


============================================================
STEP 1 - CONFIGURE MICROSOFT ENTRA ID ADMIN
============================================================

1. Open Azure Portal.
2. Search for "SQL servers".
3. Open "sc500-lab2b-sql".
4. Select:
      Settings -> Microsoft Entra ID
5. Set the Microsoft Entra administrator to:
      sc500-sql-admins
6. Select Save.
7. Confirm the Entra administrator is displayed.

![STEP 1 - Create Microsoft Entra Security Group](screenshots/01-create-entra-security-group.jpg)

RESULT:
SQL administration is controlled through Microsoft Entra ID.


============================================================
STEP 2 - REMOVE THE BROAD FIREWALL EXCEPTION
============================================================

1. Open:
      sc500-lab2b-sql
2. Select:
      Security -> Networking
3. Open the Public access section.
4. Remove/disable:
      Allow Azure services and resources to access this server
5. Select Save.

RESULT:
Azure-hosted services no longer receive a broad firewall bypass.


============================================================
STEP 3 - CREATE THE PRIVATE ENDPOINT
============================================================

1. From the SQL server, select:
      Networking -> Private access
2. Select Create private endpoint.
3. Select the lab subscription.
4. Select the lab resource group.
5. Configure the private endpoint to use:
      VNet: sc500-lab2b-vnet
6. Select the appropriate lab subnet.
7. Complete the deployment.
8. Wait until deployment shows:
      Deployment succeeded

![STEP 3 - Create Private Endpoint](screenshots/02-create-private-endpoint.jpg)

RESULT:
The SQL server has private connectivity through the lab VNet.


============================================================
STEP 4 - DISABLE PUBLIC NETWORK ACCESS
============================================================

1. Return to:
      sc500-lab2b-sql -> Networking
2. Select:
      Public access
3. Set:
      Public network access -> Disable
4. Select Save.
5. Confirm Public network access is Disabled.

![STEP 4 - Disable Public Network Access](screenshots/04-disable-public-access.jpg)

RESULT:
The SQL server is no longer accessible through its public endpoint.


============================================================
STEP 5 - ENABLE SQL AUDITING
============================================================

1. Open:
      sc500-lab2b-sql
2. Select:
      Security -> Auditing
3. Set:
      Enable Azure SQL Auditing -> On
4. Under Audit log destination select:
      Log Analytics
5. Select the lab subscription.
6. Select:
      sc500-lab2b-log
7. Select Save.
8. Confirm:
      Successfully saved server Auditing settings

![STEP 5 - Enable SQL Auditing](screenshots/03-enable-sql-auditing.jpg)

RESULT:
SQL database activity is sent to Log Analytics for monitoring
and investigation.


============================================================
STEP 6 - GENERATE AN AUDITABLE SQL EVENT
============================================================

1. Open:
      sc500-lab2b-sql
2. Select:
      Settings -> SQL databases
3. Select:
      ai-workload-db
4. Select:
      Query editor (preview)
5. Authenticate using the lab SQL credentials.
6. If Azure asks to add your client IP:
      Select Add client IP
      Select OK
      Authenticate again.
7. Run:

      SELECT * FROM AiModelMetadata;

8. Confirm that 3 rows are returned.

IMPORTANT:
Complete this step BEFORE disabling public access again.

RESULT:
The SELECT statement generates a SQL audit event.


============================================================
STEP 7 - DISABLE PUBLIC ACCESS AFTER THE TEST
============================================================

1. Return to:
      sc500-lab2b-sql -> Networking
2. Select:
      Public access
3. Set:
      Public network access -> Disable
4. Select Save.

EXPECTED RESULT:
The portal Query Editor can no longer connect.

This is EXPECTED.

The database is now intended to be accessed through
the private network.


============================================================
STEP 8 - VERIFY SQL AUDIT LOGS
============================================================

1. Open Azure Portal.
2. Search for:
      Log Analytics workspaces
3. Open:
      sc500-lab2b-log
4. Select:
      Logs
5. Switch to:
      KQL mode
6. Run:

AzureDiagnostics
| where Category == "SQLSecurityAuditEvents"
| where statement_s contains "AiModelMetadata"
| project TimeGenerated, server_instance_name_s, database_name_s, statement_s, client_ip_s
| order by TimeGenerated desc

7. Select Run.
8. Confirm the audit event containing:

      SELECT * FROM AiModelMetadata

![STEP 8 - Verify SQL Audit Logs with KQL](screenshots/05-verify-audit-log-kql.jpg)

EXPECTED RESULT:
The SQL query appears in Log Analytics.

WAIT TIME:
Normally 2-5 minutes.
Under heavy load, ingestion can take up to 10 minutes.


============================================================
STEP 9 - ENABLE MICROSOFT DEFENDER FOR DATABASES
============================================================

1. In Azure Portal search for:
      Microsoft Defender for Cloud
2. Select:
      Environment settings
3. Expand the lab subscription.
4. Select the subscription.
5. Select:
      Defender plans
6. Locate:
      Databases
7. Select:
      Select types
8. Set:
      Azure SQL Databases -> On
9. Select Continue.
10. Select Save.

RESULT:
Microsoft Defender for Databases is enabled.


============================================================
STEP 10 - VERIFY MICROSOFT DEFENDER FOR SQL
============================================================

1. Return to:
      sc500-lab2b-sql
2. Under Security select:
      Microsoft Defender for Cloud
3. Confirm:

      Microsoft Defender for SQL
      Status: ON

4. If the server initially shows "Not protected":
      Wait approximately 5 minutes.
      Refresh the page.
5. Confirm the status changes to:
      On / Protected


============================================================
FINAL VERIFICATION
============================================================

[✓] Microsoft Entra administrator configured
[✓] sc500-sql-admins configured
[✓] Broad Azure-services firewall exception removed
[✓] Private Endpoint deployed
[✓] Public network access disabled
[✓] SQL Auditing enabled
[✓] sc500-lab2b-log selected as audit destination
[✓] AiModelMetadata query executed
[✓] 3 database rows returned
[✓] Audit event generated
[✓] Audit event verified in Log Analytics
[✓] Defender for Databases enabled
[✓] Azure SQL Databases protected
[✓] Microsoft Defender for SQL = ON


============================================================
TROUBLESHOOTING
============================================================

PROBLEM:
Audit query returns 0 results.

ACTION:
1. Confirm SQL Auditing is ON.
2. Confirm destination is:
      sc500-lab2b-log
3. Wait 2-5 minutes.
4. Run the KQL query again.
5. If necessary, wait up to 10 minutes.

PROBLEM:
Query Editor stops connecting.

ACTION:
This is expected after Public network access is disabled.
Do NOT re-enable public access unless the lab specifically requires it.

PROBLEM:
Defender shows "Not protected".

ACTION:
Wait approximately 5 minutes and refresh.
Defender resource discovery is asynchronous.

PROBLEM:
"Open-source relational databases" fails to save.

ACTION:
Do not change the lab configuration to fix this.
The lab requirement is Azure SQL Databases.
Confirm that Azure SQL Databases was successfully enabled.


============================================================
FINAL SECURITY STATE
============================================================

IDENTITY
Microsoft Entra ID controls SQL administration.

NETWORK
Private Endpoint provides private connectivity.
Public network access is disabled.

AUDITING
SQL activity is captured and forwarded to Log Analytics.

MONITORING
KQL is used to verify database activity.

THREAT DETECTION
Microsoft Defender for SQL provides protection against
SQL injection, brute-force activity, anomalous access,
and other database threats.

BUSINESS BENEFIT
The database is protected through identity-based administration,
network isolation, centralized audit logging, and continuous
threat detection.

============================================================
END OF RUNBOOK
============================================================
