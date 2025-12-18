
# Scheduled Email List Documentation

## Overview
The **Scheduled Email List** feature provides an interface to retrieve and manage scheduled emails, allowing users to view, filter, and manage emails that are set to be sent at a later time. The feature interacts with the backend database to fetch scheduled emails based on various filters such as user ID, company ID, contact ID, status, etc. The email details include subject, schedule time, file count, status, and any errors encountered during scheduling.

This feature also allows users to update the scheduled email’s time or associated details using the `SaveSchedule` API endpoint, providing flexibility in managing the sending of emails.

---

## DFD (Data Flow Diagram)
```mermaid
flowchart TB
    User(("User")) e1@--> req["Sends request to get scheduled emails"]
    req e2@--> API(["API"])
    API e3@--> fetch["Fetches data from DB based on filters"] & resp["Sends scheduled emails data to User"]
    fetch e4@--> DB["Database"]
    DB e5@--> ret["Returns scheduled emails list"]
    ret e6@--> API
    resp e8@--> User
    req@{ shape: rect}
    fetch@{ shape: rect}
    resp@{ shape: rect}
    DB@{ shape: cyl}
    ret@{ shape: rect}
     req:::process
     fetch:::process
     resp:::process
     ret:::process
    classDef process fill:#e1f5ff,stroke:#0882af,stroke-width:2px
    style User fill:transparent,color:#000000
    style req color:#FFFFFF,fill:#424242,stroke:#000000
    style fetch color:#FFFFFF,stroke:#000000,fill:#424242
    style resp color:#FFFFFF,fill:#424242,stroke:#000000
    style ret fill:#424242,color:#FFFFFF,stroke:#000000
    e1@{ animation: fast }
    e2@{ animation: fast }
    e3@{ animation: fast }
    e4@{ animation: fast }
    e5@{ animation: fast }
    e6@{ animation: fast }
    e8@{ animation: fast }
```
---

## Process Flow
```mermaid
flowchart LR
    %% Step 1: User Requests
    user@{ shape: circle, label: "User" }
    request@{ shape: rect, label: "Request Scheduled Emails\n(Filter: User/Company/Status/...)" }
    user r1@-->|"Request"| request
    %% Step 2: API applies filters
    api@{ shape: rect, label: "API Filters Data\n(with Filters)" }
    request r2@-->|"To API\nwith filters"| api
    %% Step 3: DB Query
    db@{ shape: cyl, label: "Database Query & Aggregate" }
    api r3@-->|"Query/Count Matching Records"| db
    %% Step 4: Data Formatting
    format@{ shape: rect, label: "Format Data\n(Fields/Timestamps\nConvert to Practice Timezone)" }
    db r4@-->|"Results"| format
    %% Step 5: Return Data
    returnData@{ shape: rect, label: "Return Data\n(Scheduled Emails\n+ Metadata)" }
    format r5@-->|"Formatted Data"| returnData
    returnData r6@-->|"To User"| user
    %% Step 6: Email Scheduling Update
    updateAPI@{ shape: rect, label: "SaveSchedule API\n(Update/Reschedule Email)" }
    user r7@-->|"Modify/Reschedule"| updateAPI
    updateAPI r8@-->|"Update DB"| db
    %% Animation for all edges
    r1@{ animation: fast }
    r2@{ animation: fast }
    r3@{ animation: fast }
    r4@{ animation: fast }
    r5@{ animation: fast }
    r6@{ animation: fast }
    r7@{ animation: fast }
    r8@{ animation: fast }
```
---

## ER Diagram
```mermaid
erDiagram
    UserScheduleEmail {
        string Id
        string FromUserId
        string UserName
        DateTime ScheduleDateTime
        string Subject
        string Message
        string EmailId
        string ContactIds
        string CompanyIds
        string Status
        string LastError
    }

    UserScheduleEmail ||--o| User : "CreatedBy"
    UserScheduleEmail ||--o| User : "UpdatedBy"
    UserScheduleEmail ||--o| FileNameModel : "DraftFiles"
    UserScheduleEmail ||--o| EmailSentRequest : "EmailDetails"
    UserScheduleEmail ||--o| EmailSentRequestInvoice : "InvoiceDetails"
    UserScheduleEmail ||--o| EmailSentRequestInvoice : "QuoteDetails"
    UserScheduleEmail ||--o| IdNameModel : "TaskDetails"
```
---

## Entity Definition

### `UserScheduleEmail`
- **Id**: The unique identifier for the scheduled email.
- **FromUserId**: The user who drafted the email.
- **UserName**: The name of the user who drafted the email.
- **ScheduleDateTime**: The date and time when the email is scheduled to be sent.
- **Subject**: The subject of the email.
- **Message**: The body content of the email.
- **EmailId**: The email ID associated with the scheduled email.
- **ContactIds**: List of contact IDs associated with the email.
- **CompanyIds**: List of company IDs associated with the email.
- **Status**: The current status of the scheduled email (e.g., PendingSend, Sent, Failed).
- **LastError**: Any error message encountered during the scheduling process.
- **DraftFiles**: A list of files attached to the email draft.

### `UserScheduleEmailStatus`
- **PendingSend**: The email is scheduled but not yet sent.
- **Sent**: The email has been successfully sent.
- **Failed**: The email failed to send.

---

## Authentication / APIs

### Authentication
The API requires authentication via user credentials. This is handled using OAuth or JWT (JSON Web Token) to ensure secure access to the scheduled emails data.

### API Endpoints
- **GET** `/GetScheduleMails`
  - Retrieves a paginated list of scheduled emails based on the provided filters (e.g., status, user ID, company ID).
  - **Query Parameters**:
    - `start`: The starting record for pagination.
    - `length`: The number of records to fetch.
    - `search`: A search term to filter emails by subject, username, or email ID.
    - `userId`, `companyId`, `contactId`: Filters for specific user, company, or contact.
    - `status`: Filter for the email's status (e.g., pending, sent, failed).

- **POST** `/SaveSchedule/{id}`
  - Allows users to update an existing scheduled email’s details, including the scheduled send time.
  - **Body Parameters**:
    - `datetime`: The new scheduled date and time.
    - `model`: The updated email details (e.g., recipients, subject, message).

---

## Testing Guide

### Test Cases for **GET** `/GetScheduleMails`
1. **Test for valid status filter**:
   - Query with `status=0` to retrieve emails with pending status.
   - Ensure the correct emails are returned.
   
2. **Test for search functionality**:
   - Query with a search term like "Invoice" and ensure the emails returned contain that keyword in the subject or body.
   
3. **Test for pagination**:
   - Test by setting `start=0` and `length=10`, then check if the result returns only 10 records.
   
4. **Test for invalid filters**:
   - Query with an invalid user ID and ensure an empty list is returned.

### Test Cases for **POST** `/SaveSchedule/{id}`
1. **Test for updating the schedule**:
   - Send a valid request to update the schedule time and check if the scheduled time is correctly updated in the database.
   
2. **Test for invalid email data**:
   - Attempt to send an email without a subject and verify that the API returns an error message.

---

## References
- **MongoDB Aggregation Framework**: https://docs.mongodb.com/manual/core/aggregation/
- **JSON Web Tokens**: https://jwt.io/introduction/
- **ASP.NET Core Authentication**: https://docs.microsoft.com/en-us/aspnet/core/security/authentication/
