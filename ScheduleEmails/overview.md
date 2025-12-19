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
   %% External Entity
User[(User)]

%% Authentication
P1((Authenticate User))
AuthService[Authentication Auth Service]

    %% Main Processes
    CreateEmail p2@-->|"Email Data"| Database
    SaveDraft p3@-->|"Draft Email"| Database
    ScheduleEmail p4@-->|"Scheduled Time"| Service
    SendEmail p5@-->|"Sent Email Data"| Database
    TrackFailures p6@-->|"Failure Details"| FailureLog

    %% Data Flow
User edge1@--> P1
P1 edge2@--> AuthService
AuthService edge3@--> P1
    P1 df1@-->|"Create Email"| CreateEmail
    P1 df2@-->|"Save Draft"| SaveDraft
    P1 df3@-->|"Send Email"| SendEmail
    P1 df4@-->|"View Sent/Scheduled Emails"| Database
    SendEmail df5@-->|"Store Sent Email Data"| Database
    SendEmail df6@-->|"Send Scheduled Email"| ScheduleEmail
    ScheduleEmail df7@-->|"Process Scheduled Email"| Service
    Database df8@-->|"Track Failed Emails"| TrackFailures
    Database df10@-->|"Track Failed Emails"| TrackFailures

    %% Data Stores
    Database@{ shape: cyl }
    FailureLog@{ shape: cyl }
    CreateEmail@{ shape: rect }
    SaveDraft@{ shape: rect }
    ScheduleEmail@{ shape: rect }
    SendEmail@{ shape: rect }
    TrackFailures@{ shape: rect }
    AuthService@{ shape: rect }
    User@{ shape: cyl }

    %% Edge Animations
 edge1@{ animation: fast }
  edge2@{ animation: fast }
 edge3@{ animation: fast }
    P1@{ animation: fast }
    p2@{ animation: fast }
    p3@{ animation: fast }
    p4@{ animation: fast }
    p5@{ animation: fast }
    p6@{ animation: fast }
    df1@{ animation: fast }
    df2@{ animation: fast }
    df3@{ animation: fast }
    df4@{ animation: fast }
    df5@{ animation: fast }
    df6@{ animation: fast }
    df7@{ animation: fast }
    df8@{ animation: fast }
    df10@{ animation: fast }

```
---
## **Process Flow**

1. **Start**  
   - The email management process begins when a user initiates the system to compose, manage, or track emails.

2. **Compose Email**  
   - A user can compose a new email by specifying the recipients, subject, body, and attachments.
   - The user enters all necessary email details in the composition interface.

3. **System Validates Email**  
   - The system validates the email content, checking for required fields, proper email formats, and any content restrictions.
   - If validation fails, the system displays error messages to the user.

4. **Show Errors**  
   - When validation fails, specific error messages are displayed to guide the user in correcting the issues.
   - The user is redirected back to the composition screen to fix the errors and resubmit.

5. **Send Now Decision**  
   - After successful validation, the user decides how to proceed with the email:
     - Send immediately
     - Schedule for later delivery
     - Save as draft for future editing

6. **Email Sent (Immediate)**  
   - If the user chooses to send immediately, the email is marked as sent and stored in the database.
   - The email is then passed to the SMTP service for delivery.

7. **Schedule Email**  
   - Users can schedule emails to be sent at a specified time in the future.
   - Scheduled emails are stored in the database with their designated send time and processed by a background job to be sent at the scheduled time.

8. **Save as Draft**  
   - Emails can be saved as drafts for future editing. This allows users to complete their emails at a later time.
   - Drafts are stored in the database and can be retrieved for editing before sending.

9. **Track Status**  
    - The system tracks the email's status throughout the delivery process (sent, failed, pending).
    - Status updates are monitored in real-time and recorded in the database.

10. **Log Failure & Retry**  
    - Failed email attempts are logged with detailed error information, and users are notified of any issues.
    - The system retries sending emails that fail under certain conditions, implementing automatic retry logic.
    - After retry attempts, the status is updated and tracked again.

11. **List (Sent, Scheduled, Failed, Drafts)**  
    - Users can view their sent, scheduled, failed, and draft emails in a comprehensive list view.
    - The list provides an organized overview of all emails across different states in the system.

12. **Filter Emails**  
    - Emails can be filtered based on status (e.g., sent, failed, pending, scheduled, drafts) and other criteria (e.g., recipient, subject, date).
    - The filtering system allows users to quickly locate specific emails within their collection.

13. **Edit Draft**  
    - Users can edit drafts before sending them, including modifying the recipients, subject, message body, or attachments.
    - After editing, the draft returns to the composition stage for validation and processing.

14. **End**  
    - The process concludes when the user exits the email management interface or completes their email management tasks.
---

### **Process Flow Diagram**:
```mermaid
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

### **User**
Represents the system user interacting with emails.

- **user_id** (string): Unique identifier for the user
- **email_address** (string): User's email address
- **name** (string): User's full name
- **role** (string): User's role (ADMIN, MANAGER, STAFF)

### **Email**
Represents the email message.

- **email_id** (string): Unique identifier for the email
- **subject** (string): Email subject line
- **body** (text): Email body content
- **date** (datetime): Date and time the email was sent/received
- **importance** (string): Priority level (Low, Normal, High)
- **is_read** (boolean): Whether the email has been read
- **folder_id** (string): Foreign key to the folder
- **user_id** (string): Foreign key to the user

### **Draft Email**
Represents a saved but unsent email draft.

- **draft_id** (string): Unique identifier for the draft email
- **from_user_id** (string): Foreign key to the user sending the draft
- **to** (string): Recipients of the draft email
- **cc** (string): Carbon copy recipients
- **bcc** (string): Blind carbon copy recipients
- **subject** (string): Subject of the draft email
- **message** (text): Content of the draft email
- **draft_files** (string): List of files associated with the draft email

### **Scheduled Email**
Represents an email that is scheduled to be sent at a later time.

- **schedule_id** (string): Unique identifier for the scheduled email
- **from_user_id** (string): Foreign key to the user scheduling the email
- **to** (string): Recipients of the scheduled email
- **cc** (string): Carbon copy recipients
- **bcc** (string): Blind carbon copy recipients
- **subject** (string): Subject of the scheduled email
- **schedule_datetime** (datetime): Date and time the email is scheduled to be sent
- **status** (string): Status of the scheduled email (e.g., Pending, Sent, Failed)

### **Failure Log**
Stores information about failed email attempts.

- **failure_id** (string): Unique identifier for the failure log
- **email_id** (string): Foreign key to the email that failed
- **failure_reason** (string): Reason for failure (e.g., "Server error", "Invalid email address")

### **Email Thread**
Represents a conversation thread of emails.

- **thread_id** (string): Unique identifier for the email thread
- **subject** (string): Subject of the email thread
- **emails** (list): List of email messages in the thread, linked to the `USER_EMAIL` entity


---

## **Authentication / APIs**

### **Authentication**

- **Role-Based Access Control (RBAC)**: The **Email Module** uses RBAC for authentication and authorization. Access to this module is restricted using `[Authorize(Roles = "ADMIN,MANAGER,STAFF")]`, ensuring that only users with **ADMIN**, **MANAGER**, or **STAFF** roles can access it.

### **API Endpoints**

| **Description**                         | **HTTP Method** | **Endpoint**                                                                 |
|-----------------------------------------|-----------------|-----------------------------------------------------------------------------|
| **Send Email**                          | POST            | [/SendEmail](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Get Canned Messages**                 | GET             | [/CannedMessages](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Add New Draft**                       | POST            | [/SaveDraft](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Update Draft**                        | POST            | [/SaveDraft/{id}](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Get Draft Emails**                    | GET             | [/GetDraftMails](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Download Draft File**                 | GET             | [/DraftFileDownload](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Get Schedule Emails**                 | GET             | [/GetScheduleMails](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Add Schedule**                        | POST            | [/SaveSchedule](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Update Schedule**                     | POST            | [/SaveSchedule/{id}](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Update Schedule**                     | POST            | [/UpdateSchedule/{id}](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Get Schedule by ID**                  | GET             | [/schedule/{id}](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Send Scheduled Email**                | POST            | [/SendSchedule/{id}](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |
| **Download Attachment**                 | GET             | [/ScheduleFileDownload](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM) |


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

- **API Documentation**: [Link to email service API documentation].

---


