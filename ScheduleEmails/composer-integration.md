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

## **Process Flow**
```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#181818",
    "primaryColor": "#222",
    "primaryTextColor": "#EEEEEE",
    "nodeBorder": "#888",
    "fontSize": "16px"
  }
}}%%
flowchart TD
    start@{ shape: circle, label: "Start" }
    CE@{ shape: rect, label: "Compose Email" }
    VAL@{ shape: rect, label: "System Validates Email" }
    SEND@{ shape: diamond, label: "Send Now?" }
    ERR@{ shape: rect, label: "Show Errors" }
    DR@{ shape: rect, label: "Save as Draft" }
    EDIT@{ shape: rect, label: "Edit Draft" }
    SCH@{ shape: rect, label: "Schedule Email" }
    DB@{ shape: cyl, label: "Store Scheduled Email" }
    SEND2@{ shape: rect, label: "Send Email" }
    SMTP@{ shape: rect, label: "Email Service" }
    STATUS@{ shape: rect, label: "Track Status" }
    LIST@{ shape: rect, label: "List (Sent, Scheduled, Failed, Drafts)" }
    FILTER@{ shape: rect, label: "Filter Emails" }
    LOG@{ shape: rect, label: "Log Failure & Retry" }
    END@{ shape: dbl-circ, label: "End" }
    start s1@--> CE
    CE s2@--> VAL
    VAL s3@-->|Valid| SEND
    VAL s4@-->|Invalid| ERR
    ERR s5@--> CE
    SEND s6@-->|No, Save as Draft| DR
    DR s7@--> EDIT
    EDIT s8@--> CE
    SEND s9@-->|No, Schedule| SCH
    SCH s10@--> DB
    SEND s11@-->|Yes| SEND2
    SEND2 s12@--> SMTP
    SMTP s13@--> STATUS
    STATUS s14@--> LIST
    LIST s15@--> FILTER
    FILTER s16@-.-> EDIT
    STATUS s17@-->|Failed| LOG
    LOG s18@--> STATUS
    LIST s19@--> END
    %% Animated Edges
    s1@{ animation: fast }
    s2@{ animation: fast }
    s3@{ animation: fast }
    s4@{ animation: fast }
    s5@{ animation: fast }
    s6@{ animation: fast }
    s7@{ animation: fast }
    s8@{ animation: fast }
    s9@{ animation: fast }
    s10@{ animation: fast }
    s11@{ animation: fast }
    s12@{ animation: fast }
    s13@{ animation: fast }
    s14@{ animation: fast }
    s15@{ animation: fast }
    s16@{ animation: fast }
    s17@{ animation: fast }
    s18@{ animation: fast }
    s19@{ animation: fast }
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

## **Version and Change Log**

| Version | Date       | Changes                                                                                                                                          |
|---------|------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| 1.0     | 2025-12-18 | Initial version of the **Email Module** documentation created.                                                                                  |
| 1.1     | 2025-12-20 | Added API details and testing guidelines for email service interactions.                                                                         |

---

This documentation serves as a comprehensive guide for the **Email Module**, outlining its functionality, integration points, and best practices for use and testing.

