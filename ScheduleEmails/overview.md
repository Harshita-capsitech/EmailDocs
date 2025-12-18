# Email Module Documentation

---

## **Overview**

The **Email Module** is responsible for handling all email-related operations within the system, including email creation, editing, scheduling, sending, and tracking email statuses (e.g., sent, failed, pending). The module also allows users to manage drafts, view sent emails, and track failures. The Email Module integrates with other system components like the CRM, task management, and document storage systems to facilitate email-related activities.

---

## **DFD (Data Flow Diagram)**
```mermaid
flowchart TD
    %% External Entities
    User u1@-->|"Create/Send/Edit/View Emails"| EmailModule
    EmailService es1@-->|"Send Emails"| EmailModule
    Database db1@-->|"Store Email Data"| EmailModule
    FailureLog fl1@-->|"Log Failed Emails"| EmailModule
    %% Processes
    CreateEmail p1@-->|"Email Data"| EmailModule
    SaveDraft p2@-->|"Draft Email"| EmailModule
    ScheduleEmail p3@-->|"Scheduled Time"| EmailModule
    SendEmail p4@-->|"Sent Email Data"| EmailService
    TrackFailures p5@-->|"Failure Details"| FailureLog
    %% Data Stores
    EmailModule d1@-->|"Store Email"| Database
    EmailModule d2@-->|"Store Failure Data"| FailureLog
    EmailModule d3@-->|"Retrieve Email Data"| Database
    EmailModule d4@-->|"Retrieve Failure Data"| FailureLog
    %% Data Flow
    User df1@-->|"Create Email"| CreateEmail
    User df2@-->|"Save Draft"| SaveDraft
    User df3@-->|"Schedule Email"| ScheduleEmail
    User df4@-->|"Send Email"| SendEmail
    User df5@-->|"View Sent/Scheduled Emails"| EmailModule
    EmailModule df6@-->|"Send Email to Service"| EmailService
    EmailModule df7@-->|"Retrieve Sent Emails"| Database
    EmailModule df8@-->|"Retrieve Drafts"| Database
    EmailModule df9@-->|"Track Failed Emails"| TrackFailures
    %% Edge Animations
    u1@{ animation: fast }
    es1@{ animation: fast }
    db1@{ animation: fast }
    fl1@{ animation: fast }
    p1@{ animation: fast }
    p2@{ animation: fast }
    p3@{ animation: fast }
    p4@{ animation: fast }
    p5@{ animation: fast }
    d1@{ animation: fast }
    d2@{ animation: fast }
    d3@{ animation: fast }
    d4@{ animation: fast }
    df1@{ animation: fast }
    df2@{ animation: fast }
    df3@{ animation: fast }
    df4@{ animation: fast }
    df5@{ animation: fast }
    df6@{ animation: fast }
    df7@{ animation: fast }
    df8@{ animation: fast }
    df9@{ animation: fast }
    %% Optional: make stores/circle/process shapes stand out
    Database@{ shape: cyl }
    FailureLog@{ shape: cyl }
    EmailService@{ shape: rect }
    EmailModule@{ shape: stadium }
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

1. **GET /Emails**  
   Fetch a list of emails. Supports filters such as user, team, folder, and search query.
   
2. **GET /Emails/{id}**  
   Retrieve detailed information about a specific email.

3. **POST /Emails**  
   Create a new email or send an email immediately.

4. **PUT /Emails/{id}**  
   Update an email (e.g., edit a draft or reschedule a sent email).

5. **GET /Emails/Threads/{id}**  
   Retrieve all emails in a specific thread (useful for viewing conversations).

6. **GET /Emails/InitialMessage**  
   Retrieve the initial message in an email thread (used when replying to an email).

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


