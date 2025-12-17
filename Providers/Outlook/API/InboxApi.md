# Email Inbox Integration Documentation

This document details the implementation of the Email List API, which integrates both Microsoft Outlook (Graph SDK) and Google Gmail into a unified interface.

## 1. API Overview

The endpoint fetches a paginated list of email messages based on user preferences and provider types.

- **URL:** `https://localhost:5005/emails/Inbox`
- **Method:** `GET`
- **Authentication:** Requires a valid `ApplicationUserAccessTokens` associated with the logged-in user.

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `start` | int | 0 | The starting index for pagination (Skip). |
| `length` | int | 35 | Number of items to fetch (Top). |
| `search` | string | null | Search keyword for subjects or email addresses. |
| `inboxType` | string | "inbox" | Folder ID or Name (e.g., "sentitems", or a specific Outlook Folder ID). |
| `viewType` | string | "" | Filter by status: unread, read, or all. |
| `isImportant` | bool | false | Filter for high-importance emails. |
| `categoryType` | string | null | Filter by specific Outlook categories (e.g., "Green category"). |
| `sortBy` | string | "Date" | Field to sort the results. |

> **Note on inboxType:** For Microsoft Outlook, this API is highly flexible. You can pass standard folder names (like `inbox`, `sentitems`) OR a unique Folder ID (long alphanumeric string) to retrieve messages from any specific subfolder using the same logic.

## 2. Technical Workflow

The backend follows a multi-step process to fetch and transform data:

### Step A: Provider Identification

The system retrieves the user's access token from the database and checks the `EmailProvider` property:

- **Outlook:** Uses `MicrosoftController.GetMessages` via the Microsoft Graph SDK.
- **Gmail:** Uses `GoogleController.GetMessageList` via the Google Gmail API.

### Step B: Filter Construction (Outlook Logic)

For Outlook, the system builds an OData filter string. Key logic includes:

- **Time Anchoring:** It adds a filter for `receivedDateTime` or `sentDateTime` relative to `UtcNow` to ensure data consistency.
- **Dynamic Filtering:** It conditionally appends filters for Importance, Read/Unread status, Flagged status, Mentions, and Categories.
- **Search:** Implements a `contains` query on both `subject` and `from/emailAddress/address`.

### Step C: Data Fetching & Expansion

The Graph SDK call includes:
- A `$select` statement to minimize data transfer
- An `$expand` statement to fetch attachment metadata (ID and Name) in a single request

### Step D: Thread/Reply Count (Parallel Processing)

For every message retrieved, the system:
1. Identifies the `ConversationId`
2. Executes `GetReplyCount` tasks in parallel using `Task.WhenAll` to provide thread counts without significantly slowing down the response

## 3. Request & Response Example

### Example Request

```http
GET /emails/Inbox?start=0&length=100&viewType=all&sortBy=date&inboxType=AQMkADBjZjc...
```

### Example Response Payload

The response is wrapped in a standard `ApiResponse` object.

```json
{
    "executionTime": 12.308,
    "result": {
        "totalRecords": 35,
        "items": [
            {
                "id": "AAMkADBjZjc...",
                "date": "2025-12-16T07:26:41Z",
                "from": "sneha test",
                "subject": "Smart Suggestions",
                "snippet": "Hello how are you?",
                "isRead": true,
                "hasAttachment": false,
                "important": 1,
                "replyCount": 0,
                "toRecipients": [
                    {
                        "name": "Sneha Jhawar",
                        "id": "sneha.jhawar@capsitech.com"
                    }
                ]
            }
        ],
        "continuationToken": null
    },
    "status": true
}
```

## 4. Key Logic Features

### Unified Folder Handling

As mentioned, the `inboxType` parameter is the key to folder navigation.

- **Sent Items:** If `inboxType == "sentitems"`, the logic automatically switches the sorting and filtering from `receivedDateTime` to `sentDateTime`.
- **Custom Folders:** Passing a Graph Folder ID to `inboxType` allows the system to query any specific user-created folder using the exact same code path.

### Date Normalization

The helper method `GetMessageDate` is used to pick the most relevant timestamp based on the folder type:

- **Drafts:** Uses `LastModifiedDateTime`
- **Sent:** Uses `SentDateTime`
- **Inbox:** Uses `ReceivedDateTime`

### Attachment Mapping

Unlike basic API calls that only return a boolean `hasAttachments`, this implementation maps the expanded attachment collection into a simplified `IdName` list, allowing the UI to show filenames directly in the list view.