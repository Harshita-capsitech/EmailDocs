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

    %% Data Flows
    User --> P1
    P1 --> AuthService
    AuthService --> P1

    P1 --> P2
    P2 --> ScheduleDB
    ScheduleDB --> P2

    P2 --> P3
    P3 --> ScheduleDB

    P4 --> EmailService

    P3 --> FailedScheduleDB
    FailedScheduleDB --> P5
    P5 --> EmailService

    P3 --> LogService
    LogService --> P6

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
1. **Email Scheduling**: 
   - User enters email details in the **Edit Email** component.
   - The email is submitted and processed by the **ScheduleSend** component.
   - The email is stored in the system with a scheduled time for sending.

2. **Email Sending**:
   - The **ProcessScheduleEmail** service sends the email at the scheduled time.
   - If the email fails to send, the **ProcessFailedScheduleEmails** component handles retry attempts.

3. **Draft and Reschedule Management**:
   - Emails that fail to send or are not yet sent can be rescheduled or moved to drafts using the **ScheduleEmailList** component.

4. **Notifications**: 
   - If the email fails to send, users are notified and the email status is updated.

---

## ER Diagram
- **User**: Represents the sender of the email.
- **ScheduleEmail**: Stores the details of the scheduled email, including recipients, subject, content, and scheduled time.
- **DraftEmail**: Represents an email saved as a draft before it is sent.
- **ScheduledStatus**: Tracks the current status of the scheduled email (e.g., Pending, Sent, Failed).
- **Attachment**: Represents files attached to the email.

---

## Entity Definition
- **User**: Represents the sender with properties such as email, name, and roles.
- **ScheduleEmail**: Contains details of the scheduled email such as recipients (To, Cc, Bcc), subject, message, scheduled time, and attachments.
- **DraftEmail**: Stores the draft of an email that is not yet sent.
- **ScheduledStatus**: The status of the scheduled email (Pending, Sent, Failed).
- **Attachment**: A file attached to the email (e.g., invoices, quotes).

---

## Authentication / APIs
- **Authentication**: 
  - The system uses **OAuth** for secure email sending via Gmail or Outlook.
  - The **JWT tokens** are used for authentication to interact with the email services.
  
- **APIs**:
  - **POST /SaveSchedule**: Saves the scheduled email.
  - **POST /UpdateSchedule**: Updates an existing scheduled email.
  - **GET /GetScheduleEmails**: Fetches a list of scheduled emails.
  - **POST /SendSchedule**: Sends the scheduled email through external APIs.
  - **GET /ScheduleFileDownload**: Allows users to download attachments of a scheduled email.

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
- **Capsitech FluentUI**: UI components used in the scheduling and management of emails.

---

## Version and Change Log
- **Version 1.0**: Initial release with basic email scheduling functionality.
- **Version 1.1**: Added email failure handling and retry mechanism with **ProcessFailedScheduleEmails**.
- **Version 1.2**: UI enhancements for draft management and scheduling.

---
