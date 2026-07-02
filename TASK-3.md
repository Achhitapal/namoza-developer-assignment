
---

### 2. `TASK_3_INTEGRATION.md` (Task 03: HubSpot CRM & WhatsApp API Integration written answers)
```markdown
# Task 03 - Integration Architecture & CRM Design

This document details the end-to-end integration design connecting the OrthoNow landing page to HubSpot CRM, Karix WhatsApp Business API, and Google Ads.

---

## 1. End-to-End Integration Architecture

### Selected Technology
**Custom Serverless Middleware** (e.g., AWS Lambda, Google Cloud Functions, or Vercel Serverless Functions) interacting via direct API calls. 

We choose a custom API middleware over Zapier/Make because:
1.  **Deduplication Control**: Native HubSpot integrations (and standard Zapier modules) cannot deduplicate contacts by phone number without expensive multi-step search-and-branch actions.
2.  **SLA Guarantee**: Zapier/Make introduces polling queues and run-time overhead. A serverless function runs instantaneously, fulfilling our 2-minute WhatsApp delivery SLA.
3.  **Error Buffering**: Custom code allows robust queue management (e.g., Redis/BullMQ) to catch and retry failures.

---

## 2. Interviewer Trap: HubSpot Phone Deduplication

### The Challenge
HubSpot's database model deduplicates contacts automatically using **Email Address**. It does not natively deduplicate using Phone Numbers. Since this landing page collects only Name and Phone (to maximize conversion rate), using the HubSpot Forms API or native embed would create duplicate contacts if a patient submits twice, leading to sales team confusion and fragmented medical histories.

### The Solution (Custom Search Logic)
To handle deduplication by phone number, our serverless middleware will query the HubSpot CRM Search API before performing any insert:

1.  **Search Phase**: Query the HubSpot CRM search endpoint:
    `POST https://api.hubapi.com/crm/v3/objects/contacts/search`
    Using this JSON payload:
    ```json
    {
      "filterGroups": [
        {
          "filters": [
            {
              "propertyName": "phone",
              "operator": "EQ",
              "value": "+919876543210"
            }
          ]
        },
        {
          "filters": [
            {
              "propertyName": "mobilephone",
              "operator": "EQ",
              "value": "+919876543210"
            }
          ]
        }
      ],
      "properties": ["firstname", "lastname", "phone", "mobilephone", "lifecyclestage"]
    }
    ```
2.  **Branch Phase**:
    -   **If contact is found (Search returns results)**: Use the returned `id` and update the record using:
        `PATCH https://api.hubapi.com/crm/v3/objects/contacts/{contactId}`
        We append the campaign details, update `lead_status` to `"New Enquiry"`, and append a timeline note detailing the new landing page submission.
    -   **If contact is NOT found**: Create a brand new record using:
        `POST https://api.hubapi.com/crm/v3/objects/contacts`
        Setting: `firstname` = Name, `phone`/`mobilephone` = Phone, `source` = `"Google Ads - Consultation Landing Page"`, and `lead_status` = `"New Enquiry"`.

---

## 3. Failure Points & Robust Fallback Design

### The Single Biggest Failure Point
**API Timeout or Outage of the HubSpot/Karix API during form submission.** If the HubSpot API goes down or hits rate limits, the lead is lost, and the patient receives no WhatsApp confirmation.

### Fallback Design (Queue Buffer & Retry)
1.  **Decouple Submission from Processing**: 
    The serverless API will immediately write the incoming lead into a high-availability message queue (like AWS SQS or Redis BullMQ) and return a `200 OK` success status to the client-side UI in under 200ms.
2.  **Asynchronous Worker**: 
    A separate background worker processes messages from the queue. If HubSpot or Karix is down, the worker will fail to process the lead, keeping the message in the queue.
3.  **Exponential Backoff**: 
    The queue system will retry processing failed items with exponential backoff (e.g., retrying in 30s, then 2m, 5m, 15m) up to 5 times.
4.  **Dead Letter Queue (DLQ)**: 
    If a lead fails to sync after 5 attempts, it is moved to a Dead Letter Queue. An automated script sends a Slack notification to the support team to manually import the lead from the queue backup log.

---

## 4. WhatsApp SLA: Breaks & Monitoring

### What Could Break the 2-Minute SLA?
1.  **Karix API Rate Limiting**: Sending too many messages simultaneously causes the API to rate limit (`429 Too Many Requests`).
2.  **Account Balance Depletion**: WhatsApp Business API operates on a pre-paid balance model. If the balance runs out, Karix blocks outgoing templates.
3.  **Template Disapproval**: If the WhatsApp message template is edited or flagged by Meta, sending calls will return error codes.
4.  **Database/Queue Congestion**: A backup of failed leads in the queue stalls processing of new leads.

### SLA Monitoring & Alerting Setup
To guarantee the 2-minute SLA, we implement an active **Heartbeat & Status Loop**:

1.  **State Log Database Table**: 
    Every lead enqueued creates a record in our database:
    `id` | `phone` | `lead_timestamp` | `whatsapp_status` (PENDING/DELIVERED/FAILED) | `delivery_timestamp`
2.  **Webhook Listeners**: 
    Configure Karix Webhook endpoints in our middleware. When Karix delivers the WhatsApp message, they send a delivery status callback. Our endpoint matches the phone number and updates the database row to `whatsapp_status` = `'DELIVERED'`.
3.  **Escalation Watchdog Cron Job**: 
    Run a cron script every 60 seconds (using AWS EventBridge or a scheduled worker) to inspect the database for SLA violations:
    ```sql
    SELECT * FROM leads 
    WHERE whatsapp_status = 'PENDING' 
    AND lead_timestamp < NOW() - INTERVAL '90 seconds';
    ```
4.  **Instant Alerts**: 
    If any rows are returned, the script immediately triggers:
    -   An automated SMS/PagerDuty alert to the on-call developer.
    -   An emergency fallback trigger calling a secondary WhatsApp gateway (e.g., Twilio) to deliver the message.