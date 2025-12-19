# Schedule Email Feature Documentation

## Overview
The **Schedule Email** feature allows users to schedule emails to be sent at a later time, manage drafts, and track their statuses. This functionality integrates with popular email services (Gmail, Outlook) using their APIs, enabling email delivery automation. It provides features for scheduling, rescheduling, and managing draft emails, as well as handling email failures through retry mechanisms.

### Key Functionalities:
- **Scheduling Emails**: Allows users to set a specific date and time for sending emails, along with attachments (such as invoices, quotes, engagement letters, etc.).
- **Draft Management**: Users can save emails as drafts, allowing them to review and edit them before sending them.
- **Rescheduling**: Users have the ability to reschedule the email to a different time.
- **Monitoring Scheduled Emails**: The feature provides functionality to track the status of scheduled emails (e.g., Sent, Pending, Failed).
- **Notifications on Failure**: If a scheduled email fails to send, users are notified, and the system attempts to resend it.

---

## DFD (Data Flow Diagram)
```mermaid
flowchart TD
    %% External Entities
    User[(User)]
    AuthService[Authentication Service]
    EmailService[[Send Email Service]]
    ScheduleDB[(Scheduled Email Database)]
    FailedScheduleDB[(Failed Scheduled Emails Database)]
    LogService[(Logging Service)]

    %% Processes
    P1((Authenticate User))
    P2((Fetch Scheduled Emails))
    P3((View Email Details))
    P4((Send Email))
    P5((Retry Failed Email))
    P6((View Logs))

    %% Data Flows (animated edges)
    User e1@--> P1
    P1 e2@--> AuthService
    AuthService e3@--> P1

    P1 e4@--> P2
    P2 e5@--> ScheduleDB
    ScheduleDB e6@--> P2

    P2 e7@--> P3
    P3 e8@--> ScheduleDB

    P4 e9@--> EmailService

    P3 e10@--> FailedScheduleDB
    FailedScheduleDB e11@--> P5
    P5 e12@--> EmailService

    P3 e13@--> LogService
    LogService e14@--> P6

    %% Animate edges
    e1@{ animation: fast }
    e2@{ animation: fast }
    e3@{ animation: fast }
    e4@{ animation: fast }
    e5@{ animation: fast }
    e6@{ animation: fast }
    e7@{ animation: fast }
    e8@{ animation: fast }
    e9@{ animation: fast }
    e10@{ animation: fast }
    e11@{ animation: fast }
    e12@{ animation: fast }
    e13@{ animation: fast }
    e14@{ animation: fast }

    %% Styling
    style User stroke-width:2px
    style AuthService stroke-width:2px
    style EmailService stroke-width:2px
    style LogService stroke-width:2px
    style ScheduleDB stroke-width:2px
    style FailedScheduleDB stroke-width:2px
```
---

## Process Flow
```mermaid
---
config:
  layout: dagre
---
flowchart TB
    user(("User")) e1@-- starts --> edit["Edit Email
(Enter Details)"]
    edit e2@-- submit for scheduling --> scheduleSend["Schedule Send
(Process Submission)"]
    scheduleSend e3@-- store with time --> store["Store Email
with Scheduled Time"]
    store e4@-- ready to send at scheduled time --> process["Process Schedule Email
(Service)"]
    process e5@-- trigger send --> send["Send Email"]
    send e6@-- send attempt --> status{"Email Sent?"}
    status e7@-- Yes --> finished["End"]
    status e8@-- No --> retry["Process Failed Schedule Emails
(Retry/Fail Handler)"]
    retry e9@-- retry or fail --> scheduleList["Schedule Email List
(Draft/Reschedule Mgt)"]
    scheduleList e10@-- move to drafts --> draft["Move to Draft"]
    scheduleList e11@-- reschedule --> reschedule["Reschedule Email"]
    retry e12@-- notify + update status --> notif["Notify User
Update Status"]
    notif e13@-- acknowledge --> finished
    edit@{ shape: rect}
    scheduleSend@{ shape: rect}
    store@{ shape: cyl}
    process@{ shape: rect}
    send@{ shape: rect}
    finished@{ shape: dbl-circ}
    retry@{ shape: rect}
    scheduleList@{ shape: rect}
    draft@{ shape: rect}
    reschedule@{ shape: rect}
    notif@{ shape: rounded}
     edit:::core
     scheduleSend:::core
     store:::core
     process:::core
     send:::core
     scheduleList:::core
     draft:::core
     reschedule:::core
    e1@{ animation: fast }
    e2@{ animation: fast }
    e3@{ animation: fast }
    e4@{ animation: fast }
    e5@{ animation: fast }
    e6@{ animation: fast }
    e7@{ animation: fast }
    e8@{ animation: fast }
    e9@{ animation: fast }
    e10@{ animation: fast }
    e11@{ animation: fast }
    e12@{ animation: fast }
    e13@{ animation: fast }
```
---

## ER Diagram
```mermaid
erDiagram
    direction TB

    USER:::darknode {
        int userId PK
        string name
    }
    SCHEDULEEMAIL:::darknode {
        int scheduleEmailId PK
        string subject
        string recipients
        string content
        datetime scheduledTime
        string draftRefId
        string from
        int fileCount
        string lastError
        string status
    }
    DRAFTEMAIL:::darknode {
        int draftEmailId PK
        string subject
        string recipients
        string content
        datetime saveTime
        string from
        int fileCount
    }
    SCHEDULEDSTATUS:::darknode {
        int statusId PK
        string status
    }
    ATTACHMENT:::darknode {
        int attachmentId PK
        string filename
        string filetype
    }
    CONTACTS:::darknode {
        int contactId PK
        string contactName
    }
    COMPANY:::darknode {
        int companyId PK
        string companyName
    }
    EMAILID:::darknode {
        int emailId PK
        string emailContent
    }
    SCHEDULEDATETIME:::darknode {
        string followUpDate
        string followUpHour
        string followUpMinute
        string followUpTT
        string timezone
    }

    USER ||--o{ SCHEDULEEMAIL : "schedules"
    USER ||--o{ DRAFTEMAIL : "has draft"
    SCHEDULEEMAIL }o--|| SCHEDULEDSTATUS : "status"
    SCHEDULEEMAIL ||--o{ ATTACHMENT : "has"
    DRAFTEMAIL ||--o{ ATTACHMENT : "has"
    SCHEDULEEMAIL ||--o{ CONTACTS : "related to"
    SCHEDULEEMAIL ||--o{ COMPANY : "related to"
    SCHEDULEEMAIL ||--o{ EMAILID : "has"
    SCHEDULEEMAIL ||--o{ SCHEDULEDATETIME : "schedules at"
    DRAFTEMAIL ||--o{ CONTACTS : "related to"
    DRAFTEMAIL ||--o{ COMPANY : "related to"
    DRAFTEMAIL ||--o{ EMAILID : "has"
    DRAFTEMAIL ||--o{ SCHEDULEDATETIME : "schedules at"

    
```
---

## Entity Definition
- **User**: Represents the sender with properties such as email, name, and roles.
- **ScheduleEmail**: Contains details of the scheduled email such as recipients (To, Cc, Bcc), subject, message, scheduled time, and attachments.
- **DraftEmail**: Stores the draft of an email that is not yet sent.
- **ScheduledStatus**: The status of the scheduled email (Pending, Sent, Failed).
- **Attachment**: A file attached to the email (e.g., invoices, quotes).

---

## Authentication / APIs

### **Authentication**

- **Role-Based Access Control (RBAC)**: The **Email Module** uses RBAC for authentication and authorization. Access to this module is restricted using `[Authorize(Roles = "ADMIN,MANAGER,STAFF")]`, ensuring that only users with **ADMIN**, **MANAGER**, or **STAFF** roles can access it.

  
| **Description**                          | **HTTP Method** | **Endpoint URL**                                                                 |
|------------------------------------------|-----------------|----------------------------------------------------------------------------------|
| **Save the scheduled email**             | POST            | [POST /SaveSchedule](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM#/SaveSchedule) |
| **Update an existing scheduled email**   | POST            | [POST /UpdateSchedule](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM#/UpdateSchedule) |
| **Fetch a list of scheduled emails**     | GET             | [GET /GetScheduleEmails](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM#/GetScheduleEmails) |
| **Send the scheduled email through external APIs** | POST            | [POST /SendSchedule](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM#/SendSchedule) |
| **Download attachments of a scheduled email** | GET             | [GET /ScheduleFileDownload](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM#/ScheduleFileDownload) |


---

## Testing Guide
- **Unit Testing**: Tests for individual components like **ScheduleSend** and **ProcessScheduleEmail** to ensure email scheduling functionality works as expected.
- **Integration Testing**: Tests interaction with external email APIs (Gmail, Outlook) to ensure emails are sent at the correct time.
- **Error Handling Testing**: Tests for email failures and proper retry mechanisms handled by **ProcessFailedScheduleEmails**.
- **UI Testing**: Verifies the **ScheduleEmailList** component works for scheduling, rescheduling, and managing drafts.

---

## References
- **EmailService**: Core service that interacts with Gmail, Outlook, and other external email APIs.
- **OAuth Authentication**: OAuth 2.0 for secure email service access.

---






