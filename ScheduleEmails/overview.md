# Email Module Documentation

---

## **Overview**

The **Email Module** serves as the backbone of communication within the system, managing a comprehensive range of email-related operations with unmatched efficiency and precision. It handles everything from the seamless creation and editing of emails to the sophisticated scheduling and timely delivery of messages. Beyond basic sending, this powerhouse module tracks email statuses meticulously—whether it's a successful delivery, a pending message, or a failed attempt—ensuring that no communication is ever lost.

Designed to meet the needs of modern businesses, the Email Module offers users full control over their email activities, allowing them to manage drafts, review sent emails, and swiftly identify any failures or delivery issues. It seamlessly integrates with key system components, such as CRM, task management, and document storage, to create a unified communication experience.

Whether you're sending invoices, client updates, or internal notifications, the Email Module empowers users with advanced features like automated retries for failed emails, detailed status tracking, and effortless email rescheduling. It acts as a central hub for all email-related tasks, ensuring that no message ever goes unnoticed or untracked.

From drafting personalized messages to automating email workflows, this module is engineered to enhance efficiency and streamline communication processes, making it indispensable for every team within the system. With its robust functionality, the Email Module ensures that email management is no longer a chore, but a seamless, integrated experience that drives productivity and fosters communication across the entire organization.


---

## **DFD (Data Flow Diagram)**
```mermaid
flowchart TD
    %% External Entities
    User u1@-->|"Create/Send/Edit/View Emails"| Database
    EmailService es1@-->|"Send Emails"| Database
    FailureLog fl1@-->|"Log Failed Emails"| Database

    %% Processes
    CreateEmail p1@-->|"Email Data"| Database
    SaveDraft p2@-->|"Draft Email"| Database
    ScheduleEmail p3@-->|"Scheduled Time"| Database
    SendEmail p4@-->|"Sent Email Data"| EmailService
    TrackFailures p5@-->|"Failure Details"| FailureLog

    %% Data Flow
    User df1@-->|"Create Email"| CreateEmail
    User df2@-->|"Save Draft"| SaveDraft
    User df3@-->|"Schedule Email"| ScheduleEmail
    User df4@-->|"Send Email"| SendEmail
    User df5@-->|"View Sent/Scheduled Emails"| Database
    Database df6@-->|"Send Email to Service"| EmailService
    Database df9@-->|"Track Failed Emails"| TrackFailures

    %% Edge Animations
    u1@{ animation: fast }
    es1@{ animation: fast }
    fl1@{ animation: fast }
    p1@{ animation: fast }
    p2@{ animation: fast }
    p3@{ animation: fast }
    p4@{ animation: fast }
    p5@{ animation: fast }
    df1@{ animation: fast }
    df2@{ animation: fast }
    df3@{ animation: fast }
    df4@{ animation: fast }
    df5@{ animation: fast }
    df6@{ animation: fast }
    df9@{ animation: fast }

    %% Shapes
    Database@{ shape: cyl }
    FailureLog@{ shape: cyl }
    EmailService@{ shape: rect }
    CreateEmail@{ shape: rect }
    SaveDraft@{ shape: rect }
    ScheduleEmail@{ shape: rect }
    SendEmail@{ shape: rect }
    TrackFailures@{ shape: rect }
    User@{ shape: circle }

```
---

## **Process Flow  (Process Flow Diagram)**
```mermaid
---
config:
  theme: base
  themeVariables:
    background: '#181818'
    primaryColor: '#222'
    primaryTextColor: '#eeeeee'
    nodeBorder: '#888'
    fontSize: 16px
  layout: dagre
---
flowchart TB
    start(["Start"]) e1@--> CE["Compose Email"]
    CE e2@--> VAL["System Validates Email"]
    VAL e3@-- Valid --> SEND{"Send Now?"}
    VAL e4@-- Invalid --> ERR["Show Errors"]
    ERR e5@--> CE
    SEND e6@-- No, Schedule --> DB["Email Sent"]
    SEND e7@-- Yes, Schedule --> SCH["Schedule Email"]
    SCH e8@--> SMTP["Service"]
    SMTP e9@--> STATUS["Track Status"]
    STATUS e10@--> LIST["List (Sent, Scheduled, Failed, Drafts)"]
    LIST e11@--> FILTER["Filter Emails"]
    LIST e18@--> END(["End"])
    FILTER e12@-.-> EDIT["Edit Draft"]
    STATUS e13@-- Failed --> LOG["Log Failure & Retry"]
    LOG e14@--> STATUS
    SEND e15@-- No, Save as Draft --> DR["Save as Draft"]
    DR e16@--> EDIT
    EDIT e17@--> CE

    style start animation:fast
    style CE animation:fast
    style VAL animation:fast
    style SEND animation:fast
    style ERR animation:fast
    style DB animation:fast
    style SCH animation:fast
    style SMTP animation:fast
    style STATUS animation:fast
    style LIST animation:fast
    style FILTER animation:fast
    style END animation:fast
    style EDIT animation:fast
    style LOG animation:fast
    style DR animation:fast

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
    e15@{ animation: fast }
    e16@{ animation: fast }
    e17@{ animation: fast }
    e18@{ animation: fast }
```
---

## **ER Diagram (Entity-Relationship Diagram)**
```mermaid
erDiagram
    USER {
        string UserId
        string UserName
        string Email
        string Role
    }
    
    USER_EMAIL {
        string EmailId
        string FromUserId
        string To
        string CC
        string BCC
        string Subject
        string Message
        string Attachments
        string Status
    }
    
    USER_DRAFT_EMAIL {
        string DraftId
        string FromUserId
        string To
        string CC
        string BCC
        string Subject
        string Message
        string DraftFiles
    }
    
    USER_SCHEDULE_EMAIL {
        string ScheduleId
        string FromUserId
        string To
        string CC
        string BCC
        string Subject
        string ScheduleDateTime
        string Status
    }
    
    FAILURE_LOG {
        string FailureId
        string EmailId
        string FailureReason
    }
    
    EMAIL_THREAD {
        string ThreadId
        string Subject
        list Emails
    }

    USER ||--o{ USER_EMAIL : sends
    USER_EMAIL ||--o{ USER_DRAFT_EMAIL : saves
    USER ||--o{ USER_SCHEDULE_EMAIL : schedules
    USER_EMAIL ||--o{ FAILURE_LOG : logs
    EMAIL_THREAD ||--o{ USER_EMAIL : contains
```

---

## **Entity Definitions**

1. **Email**: Contains the details of an email message, including the sender, recipients (To, CC, BCC), subject, message body, attachments, and status.
2. **User**: Represents users of the system, each with a specific role (Admin, Manager, Staff) that determines their access to email-related features.
3. **Draft**: An email that is saved but not sent yet. It is stored temporarily and can be modified later before being sent.
4. **Scheduled Email**: A scheduled email that is set to be sent at a later time. The system processes these emails and sends them at the scheduled time.
5. **Sent Email**: Represents an email that has been successfully dispatched to recipients.
6. **Failure Log**: Stores information about failed email attempts, such as the reason for failure, time of failure, and any error messages.

---

## **Authentication / APIs**

### **Authentication**

- **Role-Based Access Control (RBAC)**: The **Email Module** uses RBAC for authentication, meaning that different user roles have different access levels and permissions.
- **JWT Tokens**: Users must authenticate via JWT (JSON Web Tokens) to access the system's email-related services.

### **API Endpoints**

| **Description**                         | **HTTP Method** | **Endpoint**                                                                 |
|-----------------------------------------|-----------------|-----------------------------------------------------------------------------|
| **Send Email**                          | POST            | [/SendEmail](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Get Canned Messages**                 | GET             | [/CannedMessages](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Add New Draft**                       | POST            | [/SaveDraft](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Update Draft**                        | POST            | [/SaveDraft/{id}](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Get Draft Emails**                    | GET             | [/GetDraftMails](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Download Draft File**                 | GET             | [/DraftFileDownload](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Get Schedule Emails**                 | GET             | [/GetScheduleMails](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Add Schedule**                        | POST            | [/SaveSchedule](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Update Schedule**                     | POST            | [/SaveSchedule/{id}](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Update Schedule**                     | POST            | [/UpdateSchedule/{id}](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Get Schedule by ID**                  | GET             | [/schedule/{id}](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Send Scheduled Email**                | POST            | [/SendSchedule/{id}](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |
| **Download Schedule File**              | GET             | [/ScheduleFileDownload](https://emails.servicesuat.actingoffice.com/swagger/index.html](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)) |


---

## **Testing Guide**

### **Unit Testing**

- **Email Creation**: Ensure that emails can be created and saved correctly.
- **Draft Saving**: Test the ability to save emails as drafts and retrieve them.
- **Email Sending**: Test the email sending logic, including successful delivery and failure handling.

### **Integration Testing**

- **Email Service Integration**: Verify that the email service (SMTP or third-party) is integrated properly with the system for sending emails.
- **Database Integration**: Ensure that emails are correctly stored in the database and that all metadata (e.g., recipient, subject, status) is saved appropriately.

### **End-to-End Testing**

- **End-to-End Flow**: Test the complete process from email creation, saving as a draft, scheduling, and sending to failure handling and retries.
- **UI Testing**: Ensure that users can interact with the email interface as expected, including creating, editing, and managing emails.

### **API Testing**

- Use **Postman** or **Swagger** to test the various API endpoints.
- Test edge cases such as missing required fields or invalid email addresses.

---

## **References**

- **Email Service API Documentation**: [Link to email service API documentation].
- **Fluent UI Documentation**: [Link to Fluent UI documentation].
- **Capsitech Storage and Services**: [Link to Capsitech documentation].

---


