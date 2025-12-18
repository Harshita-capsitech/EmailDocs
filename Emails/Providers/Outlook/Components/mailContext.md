# Email Get Messages – Technical Documentation



## Overview

This document describes the backend implementation of the **GetMessages API**, which retrieves paginated email messages from Microsoft Outlook using the **Microsoft Graph SDK**.

The API supports Inbox, Sent Items, and Drafts through a single unified method and provides advanced filtering such as unread/read, flagged, important, attachments, mentions, categories, search, and date-based sorting.

---

## DFD (Data Flow Diagram)

```mermaid
flowchart LR
    UI[Client UI] edge1@--> MC[MailContext / EmailService]
    MC edge2@--> API[Backend GetMessages API]
    API edge3@--> GSDK[Microsoft Graph SDK]
    GSDK edge4@--> MB[Outlook Mailbox]
    MB edge5@--> GSDK
    GSDK edge6@--> API
    API edge7@--> NORM[Normalize & Enrich Data]
    NORM edge8@--> UI

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }

```

The backend acts as an abstraction layer that converts UI filters into Graph OData queries and normalizes Graph responses into application-friendly models.

---

## Process Flow

```mermaid
sequenceDiagram
    participant UI as Client UI
    participant API as Backend API
    participant G as Microsoft Graph

    UI->>API: GetMessages(filters, paging)
    API->>API: Build OData filters
    API->>G: Messages.GetAsync()
    G-->>API: MessageCollectionResponse
    API->>API: Fetch ReplyCount per Conversation
    API->>API: Normalize to EmailsListItem
    API-->>UI: PagedData + ContinuationToken
```

---

## ER Diagram

```mermaid
erDiagram
    USER ||--o{ MAILFOLDER : has
    MAILFOLDER ||--o{ MESSAGE : contains
    MESSAGE }o--|| CONVERSATION : belongs_to
    MESSAGE ||--o{ ATTACHMENT : has

    USER {
        string userId
        string email
    }

    MAILFOLDER {
        string id
        string name
    }

    MESSAGE {
        string id
        string subject
        datetime receivedDateTime
        bool isRead
        bool hasAttachments
        string conversationId
    }

    CONVERSATION {
        string conversationId
    }

    ATTACHMENT {
        string id
        string name
    }
```

---

## Entity Definition

### EmailsListItem

```mermaid
classDiagram
    class EmailsListItem {
        string Id
        string Subject
        string From
        DateTime Date
        bool IsRead
        Importance Important
        FlagStatus Flag
        bool HasAttachment
        List~IdName~ Attachments
        string[] Categories
        string ConversationId
        int ReplyCount
    }
```

| Field          | Type         | Description              |
| -------------- | ------------ | ------------------------ |
| Id             | string       | Message ID               |
| Subject        | string       | Email subject            |
| From           | string       | Sender name              |
| Date           | DateTime     | Received/Sent/Draft date |
| IsRead         | bool         | Read status              |
| Important      | Importance   | Importance level         |
| Flag           | FlagStatus   | Follow-up flag           |
| HasAttachment  | bool         | Attachment indicator     |
| Attachments    | List<IdName> | Attachment metadata      |
| Categories     | string[]     | Outlook categories       |
| ConversationId | string       | Conversation grouping    |
| ReplyCount     | int          | Replies in conversation  |

---

## Authentication / APIs

### Authentication

* Uses **Bearer Token (OAuth 2.0)**
* Token is resolved to a Microsoft identity
* `GraphServiceClient` is instantiated per authenticated user

### API Signature

```csharp
Task<PagedData<EmailsListItem>> GetMessages(...)
```

The API translates frontend filters into Graph-compatible OData filters.

---

## Testing Guide

### Unit Testing

* Filter generation logic
* Date selection logic per folder
* DTO mapping

### Integration Testing

* Microsoft Graph sandbox account
* Pagination & continuation token
* Folder switching

### Edge Cases

* Empty mailbox
* Large conversations
* Missing ConversationId

---

## References

* Microsoft Graph Messages API
* OData Query Options
* Microsoft Graph SDK for .NET
