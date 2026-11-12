# Marketing Multi-Channel Campaign Trigger



A MuleSoft-based campaign management and orchestration application that evaluates customer eligibility, personalizes campaign messages, and manages multi-channel campaign execution through REST APIs.



## Project Overview



The **Marketing Multi-Channel Campaign Trigger** automates the process of triggering marketing campaigns across Email, SMS, and Push Notification channels.



The application retrieves campaign and customer information from a relational database, checks customer consent and suppression rules, evaluates recent contact history, and determines whether each customer is eligible to receive a campaign message.



It supports three execution modes: `DRY_RUN`, `PREVIEW`, and `EXECUTE`.



## Key Features



- **REST API Integration:** Exposes campaign management and trigger endpoints using MuleSoft HTTP Listener and APIKit.

- **Campaign Validation:** Validates campaign information and approval status before processing.

- **Customer Eligibility:** Evaluates customer consent, suppression lists, and recent contact history.

- **Multi-Channel Support:** Supports Email, SMS, and Push Notification message preparation.

- **Message Personalization:** Generates personalized messages based on customer and campaign information.

- **Execution Modes:** Supports DRY_RUN, PREVIEW, and EXECUTE operations.

- **Idempotency:** Uses an idempotency key to prevent duplicate campaign execution requests.

- **Database Integration:** Stores campaign execution details, recipient decisions, and scheduled messages.

- **Scheduled Processing:** Uses a scheduler to identify due messages and publish them to internal message queues.

- **Email Preview Report:** Sends campaign preview reports to the configured marketing contact through SMTP.

- **Audit and Tracking:** Maintains execution status, eligibility decisions, and message processing records.



## Technology Stack



| Component | Technology |
|---|---|
| Integration Platform | MuleSoft Anypoint Platform |
| Runtime | Mule Runtime 4.13.2 |
| Development Environment | Anypoint Studio 7.29.1 |
| Programming / Transformation | DataWeave 2.0 |
| API Specification | RAML |
| Database | TiDB Cloud / MySQL 8.0 |
| Database Connectivity | Mule Database Connector |
| Email Integration | SMTP |
| Messaging | Mule VM Connector |
| Build Tool | Maven |
| Version Control | Git and GitHub |
| Deployment | CloudHub 2.0 |


## System Architecture



```text

             Client / PowerShell
                     |
                     v
               HTTP Listener
                     |
                     v
               APIKit Router
                     |
                     v
            Campaign Trigger Flow
                     |
          +----------+----------+
          |                     |
          v                     v
    Database Layer        DataWeave
          |                     |
          v                     v
 Campaign / Customer     Eligibility Evaluation

 Consent / Suppression   Message Personalization

 Contact History                |
          |                     |
          +----------+----------+
                     |
                     v
              Execution Modes
                     |
          +----------+----------+
          |          |          |
          v          v          v
        DRY_RUN    PREVIEW    EXECUTE
                     |          |
                     v          v
                SMTP Report  Scheduled Messages
                                |
                                v
                           Scheduler
                                |
                                v
                          Internal VM Queues
                                |
                                v
                         Queue Consumers

```



## Execution Modes



### 1. DRY_RUN



- Evaluates the campaign without performing a live execution.

- Checks customer eligibility.

- Identifies excluded customers and exclusion reasons.

- Generates execution statistics.

- Helps validate campaign configuration before execution.



### 2. PREVIEW



- Performs campaign eligibility evaluation.

- Generates personalized messages for eligible customers.

- Produces a campaign preview report.

- Sends the preview report to the configured marketing contact through SMTP.

- Does not perform actual customer-channel delivery.



### 3. EXECUTE



- Creates a campaign execution record.

- Generates personalized messages.

- Creates scheduled message records in the database.

- Uses a scheduler to identify messages that are due.

- Publishes due messages to internal VM queues.

- Tracks message processing status.



**Implementation note:** The current SMS and Push queue consumers, and the internal queue processing flow, are used for message logging and processing demonstration. Actual external SMS and Push delivery integrations are not implemented. Customer email delivery is also not currently implemented as a live campaign-delivery flow. The SMTP integration is used for preview reporting.



## API Endpoints



**Base URL (CloudHub 2.0):**



```text

https://marketing-campaign-trigger-cv0iri.5sc6y6-3.usa-e2.cloudhub.io

```



**Base Path:**



```text

/api/v1

```



### 1. Get Campaign Details



Retrieves campaign information using the campaign ID.



| Property | Value |
|---|---|
| Method | GET |
| Endpoint | `/api/v1/db-test/campaigns/{campaignId}` |


Example:



```http

GET /api/v1/db-test/campaigns/CMP-2026-0918

```



### 2. Trigger Campaign



Triggers a campaign using the selected execution mode.



| Property | Value |
|---|---|
| Method | POST |
| Endpoint | `/api/v1/campaigns/{campaignId}/trigger` |
| Required Header | `Idempotency-Key` |
| Content-Type | `application/json` |



Example request:



```http

POST /api/v1/campaigns/CMP-2026-0918/trigger

```



Request body:



```json

{

  "mode": "DRY_RUN"

}

```



Supported modes:



```text

DRY_RUN

PREVIEW

EXECUTE

```



**Important:** Supply a new unique `Idempotency-Key` for each new execution request. Reusing an existing key prevents duplicate execution.



## Database Design



The application uses a relational database to manage campaigns, customers, consent information, execution records, and scheduled messages.



Important tables include:



| Table | Purpose |
|---|---|
| `campaigns` | Stores campaign details and approval information |
| `customers` | Stores customer information |
| `customer_consents` | Maintains channel-specific customer consent |
| `suppression_list` | Stores customers excluded from campaigns |
| `customer_contact_history` | Tracks previous customer communications |
| `campaign_executions` | Maintains campaign execution records |
| `campaign_recipient_log` | Records recipient eligibility decisions |
| `campaign_scheduled_messages` | Stores scheduled personalized messages |
| `campaign_templates` | Stores campaign message templates |
| `campaign_execution_locks` | Supports execution locking |



### Eligibility Evaluation



The application evaluates customers using business rules such as:



- Customer channel consent.

- Suppression list membership.

- Recent contact history.

- Campaign approval and validity.



Customers are classified as:



- `ELIGIBLE` – Customer satisfies the applicable campaign rules.

- `EXCLUDED` – Customer does not satisfy one or more eligibility rules.



The application records exclusion reasons to support auditing and transparency.



## Project Structure



```text

marketing-campaign-trigger/
│
├── src/
│   ├── main/
│   │   ├── mule/
│   │   │   └── marketing-campaign-trigger.xml
│   │   │
│   │   └── resources/
│   │       ├── api/
│   │       │   └── marketing-campaign-trigger.raml
│   │       │
│   │       └── config.properties
│   │
│   └── test/
│       └── munit/
│           └── marketing-campaign-trigger-test-suite.xml
│
├── pom.xml
├── .gitignore
└── README.md

```



**Note:** The local `config.properties` file contains environment-specific configuration and must not be committed to GitHub.



## Prerequisites



Install the following before running the application:



- Java 17

- Anypoint Studio 7.29.1

- Maven

- MySQL 8.0 or compatible TiDB database

- Anypoint Platform account



## Local Setup and Execution



### Step 1: Clone the Repository



```bash

git clone https://github.com/RekularapuMurali/marketing-multi-channel-campaign-trigger.git

```



### Step 2: Open the Project



Import the cloned project into Anypoint Studio as an existing Mule project.



### Step 3: Configure Database and Email Properties



Create the local configuration file:



```text

src/main/resources/config.properties

```



Configure the following properties using your own credentials:



```properties

db.host=YOUR_DATABASE_HOST

db.port=YOUR_DATABASE_PORT

db.user=YOUR_DATABASE_USERNAME

db.password=YOUR_DATABASE_PASSWORD

db.database=marketing_campaign



email.username=YOUR_EMAIL_ADDRESS

email.password=YOUR_EMAIL_APP_PASSWORD

```



Do not upload real credentials, passwords, or private certificates to GitHub.



### Step 4: Configure the Database



Create or import the `marketing_campaign` database and ensure that the required tables are available.



### Step 5: Build the Application



Run:



```bash

mvn clean package

```



To build without running MUnit tests:



```bash

mvn clean package -DskipMunitTests

```



### Step 6: Run the Application



Run the Mule application from Anypoint Studio.



The local API base URL is:



```text

http://localhost:8081/api/v1

```



## Testing with PowerShell



Set the API variables:



```powershell

$baseUrl = "https://marketing-campaign-trigger-cv0iri.5sc6y6-3.usa-e2.cloudhub.io"

$campaignId = "CMP-2026-0918"



$getUrl = "$baseUrl/api/v1/db-test/campaigns/$campaignId"

$triggerUrl = "$baseUrl/api/v1/campaigns/$campaignId/trigger"

```



### Test 1: Retrieve Campaign Details



```powershell

Invoke-RestMethod -Uri $getUrl -Method GET |

    ConvertTo-Json -Depth 5

```



### Test 2: DRY_RUN



```powershell

$headers = @{

    "Idempotency-Key" = [guid]::NewGuid().ToString()

}



$body = @{

    mode = "DRY_RUN"

} | ConvertTo-Json



Invoke-RestMethod -Uri $triggerUrl `

    -Method POST `

    -Headers $headers `

    -ContentType "application/json" `

    -Body $body

```



### Test 3: PREVIEW



```powershell

$headers = @{

    "Idempotency-Key" = [guid]::NewGuid().ToString()

}



$body = @{

    mode = "PREVIEW"

} | ConvertTo-Json



Invoke-RestMethod -Uri $triggerUrl `

    -Method POST `

    -Headers $headers `

    -ContentType "application/json" `

    -Body $body

```



### Test 4: EXECUTE



```powershell

$headers = @{

    "Idempotency-Key" = [guid]::NewGuid().ToString()

}



$body = @{

    mode = "EXECUTE"

} | ConvertTo-Json



Invoke-RestMethod -Uri $triggerUrl `

    -Method POST `

    -Headers $headers `

    -ContentType "application/json" `

    -Body $body

```



## Deployment



The application is deployed on **MuleSoft CloudHub 2.0**.


| Property | Value |
|---|---|
| Deployment Platform | CloudHub 2.0 |
| Application Name | `marketing-campaign-trigger` |
| Runtime Version | Mule Runtime 4.13.2 |
| Deployment Region | US East (Ohio) |
| Replicas | 1 |



Environment-specific database and email credentials should be configured through Runtime Manager application properties rather than committed to the source repository.



## Future Enhancements



- Integrate actual SMS delivery through an SMS gateway.

- Integrate Push Notification services.

- Implement live customer email delivery.

- Add advanced campaign analytics and reporting.

- Add retry mechanisms and dead-letter queue handling.

- Improve monitoring, alerting, and failure recovery.

- Implement additional automated integration tests.

- Add authentication and authorization for campaign APIs.



## Author



**Murali Rekularapu**



BE – Information Technology

Chaitanya Bharathi Institute of Technology (CBIT), Hyderabad



GitHub: [RekularapuMurali](https://github.com/RekularapuMurali)



---



## License



This project is developed for educational and demonstration purposes.
