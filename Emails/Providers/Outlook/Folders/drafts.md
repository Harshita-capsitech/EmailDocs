# Email Messages Retrieval Module – GetMessages

## Overview

The **GetMessages** module is a backend service responsible for retrieving a paginated, filtered, and normalized list of email messages from **Microsoft Graph**. It acts as the **single backend entry point** for inbox-like views (Inbox, Sent Items, Drafts, etc.) used by frontend applications.

### Purpose

* Provide a **unified email listing API** across folders
* Support rich filtering (read/unread, flagged, categories, attachments, mentions)
* Enable efficient pagination and infinite scroll
* Normalize raw Graph responses into UI-ready models

### Problems It Solves

* Avoids multiple folder-specific APIs
* Centralizes OData filter construction
* Ensures consistent sorting and pagination
* Decouples frontend from Graph SDK complexities

---

## Data Flow Diagram (DFD)

```mermaid
flowchart LR
    UI[Frontend UI] edge1@--> API[Email Controller]
    API edge2@--> Service[GetMessages Service]
    Service edge3@--> Graph[Microsoft Graph API]
    Graph edge4@--> Service
    Service edge5@--> API
    API edge6@--> UI

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }

```

---

## Process Flow

```mermaid
flowchart TD
    Start[Request Received] edge1@--> Params[Parse Query Parameters]
    Params edge2@--> Filters[Build OData Filters]
    Filters edge3@--> GraphCall[GraphServiceClient.Messages.Get]
    GraphCall edge4@--> GraphResp[MessageCollectionResponse]
    GraphResp edge5@--> ReplyCount[Fetch Reply Count per Conversation]
    ReplyCount edge6@--> Normalize[Map to EmailsListItem]
    Normalize edge7@--> Page[Build PagedData]
    Page edge8@--> End[Return Response]

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
```

---

## ER Diagram

```mermaid
erDiagram
    USER ||--o{ MESSAGE : owns
    MESSAGE ||--o{ ATTACHMENT : contains
    MESSAGE }o--o{ CATEGORY : tagged_with
    MESSAGE ||--o{ RECIPIENT : sent_to

    USER {
        string userId
        string email
    }

    MESSAGE {
        string id
        string subject
        datetime receivedDate
        datetime sentDate
        boolean isRead
        boolean isDraft
        string conversationId
        string importance
    }

    ATTACHMENT {
        string id
        string name
    }

    CATEGORY {
        string name
    }

    RECIPIENT {
        string id
        string name
    }
```

---

## Entity Definition

### EmailsListItem

| Property         | Type               | Description             |
| ---------------- | ------------------ | ----------------------- |
| `Id`             | string             | Unique message ID       |
| `From`           | string             | Sender display name     |
| `ToRecipients`   | List<IdNameModel>  | Recipient list          |
| `Subject`        | string             | Email subject           |
| `Snippet`        | string             | Body preview            |
| `Date`           | DateTime           | Normalized message date |
| `IsRead`         | bool               | Read status             |
| `IsDraft`        | bool               | Draft indicator         |
| `Important`      | Importance         | Importance flag         |
| `Flag`           | FollowupFlagStatus | Follow-up flag status   |
| `Categories`     | List<string>       | Assigned categories     |
| `HasAttachment`  | bool               | Attachment indicator    |
| `Attachments`    | List<IdName>       | Attachment metadata     |
| `ConversationId` | string             | Conversation thread ID  |
| `ReplyCount`     | int                | Replies count           |

---

## Authentication / APIs

### Authentication

* OAuth 2.0 access token
* Token scoped for **Mail.Read** / **Mail.ReadWrite**
* Injected into `GraphServiceClient`

### External APIs

* **Microsoft Graph**: `/me/mailFolders/{folder}/messages`

### Backend API Contract

```http
GET /api/emails/messages
```

### Key Query Parameters

* `inboxType` (inbox, sentitems, drafts)
* `viewType` (unread, read, all)
* `search`
* `isFlagged`
* `categoryType`
* `hasAttachment`
* `hasMentioned`
* `start`, `length`

---

## Testing Guide

### Unit Tests

* Filter construction logic
* View type handling (read/unread)
* Date normalization logic
* Reply count aggregation

### Integration Tests

* Microsoft Graph sandbox
* Pagination & continuation token validation
* Large mailbox scenarios

### Edge Cases

* Empty folders
* Messages without ConversationId
* Draft vs Sent date resolution
* Search strings with special characters

---

## References

* Microsoft Graph Mail API Documentation
* OData Filter Syntax
* Internal Mail Architecture Guide
