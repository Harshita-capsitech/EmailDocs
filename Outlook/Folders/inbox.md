# Inbox

## Overview

The Inbox feature provides a single, consistent inbox experience while supporting **both Gmail and Outlook** behind the scenes. The UI and API remain provider-agnostic, while the backend routes each request to the correct provider implementation based on the authenticated user.

This document explains how the Inbox works end-to-end (Frontend → API → Provider routing), including paging, filters, sorting, search, and refresh behavior.

> 🔗 Related docs:  
> - [Mail Module Overview](../index.md)  
> - [Outlook Provider](./outlook.md)

---

## End-to-End Flow

### Frontend → Backend → Provider

1. The Inbox screen loads emails using `useMail()` (MailContext).
2. `MailContext.getItems()` calls `EmailService.getInboxMessages(...)`.
3. The backend endpoint `GET /Inbox` receives the request.
4. The backend reads the authenticated user (`User.GetUserId()`), loads access tokens, and detects the provider.
5. The backend routes the request:
   - Outlook → Microsoft Graph provider method
   - Gmail → Google provider method
6. The API returns a unified response model which the frontend groups and renders.

Provider routing (conceptual):
```
User Request → /Inbox API → Provider Flag Check
                            ├─→ Outlook Provider (Microsoft)
                            └─→ Gmail Provider (Gmail API)
```

---

## Frontend Implementation

### Inbox UI Component

The Inbox UI is implemented in `NewInboxEmails` and includes:

- Header + filter menu
- Toggle buttons: **Unread / Read / All**
- Search input (debounced)
- Grouped infinite list (`ControlledGroupList`)
- Right-click context menu actions

---

## View Toggles (Unread / Read / All)

The toggle is controlled through `listParams.inboxViewType`:

- `unread`
- `read`
- `all`

When a toggle changes:

- `selectedMail` is cleared
- `nextPageToken` is reset
- emails are refetched for the new view

---

## Search

Search is implemented using a debounced input (500ms).  
When the folder changes, search is reset.

Behavior:

- User types in search box
- `updateParams({ search: value, nextPageToken: undefined })` is invoked (debounced)
- `onSearch()` can trigger a fetch using the updated search term

---

## Grouped Infinite List

Inbox uses a grouped list renderer (`ControlledGroupList`).  
Emails are grouped according to the active sort grouping logic (usually by date buckets like Today/Yesterday).

Infinite scroll behavior:

- `onLoadMore` calculates the start offset using `getUniqueItemCount(...)`
- calls `onScroll(start, PAGE, 'inbox')`
- backend returns more items using continuation tokens (if available)

---

## Context Menu Actions

Right-clicking an email opens a contextual menu with:

### Archive
- Calls `EmailService.moveFolder(id, 'archive')`
- Updates local cache via `updateEmails(..., EmailUpdateTypes.archive)`

### Delete
- Calls `EmailService.moveFolder(id, 'deleteditems')`
- Updates local cache via `updateEmails(..., EmailUpdateTypes.archive)` (current code treats delete similarly)

### Mark Read / Unread
- Calls `EmailService.updateMessage(id, { isRead, type: ReadUnRead })`
- Updates local cache via `updateEmails(..., EmailUpdateTypes.readUnread)`

### Flag
Supports presets and explicit updates:

- Today
- Tomorrow
- This week
- Next week
- Mark complete
- Clear flag

Calls:
- `EmailService.updateMessage(id, { flagModel, type: Flag })`

Updates local cache via:
- `updateEmails(..., EmailUpdateTypes.flag)`

### Categories
- Assign category → `EmailService.updateMessage(id, { categoryId, type: Category })`
- Clear category → `EmailService.updateMessage(id, { categoryId: undefined, type: Category })`

Updates local cache via:
- `updateEmails(..., EmailUpdateTypes.category)`

---

## Folder-Specific View Logic

The fetch logic chooses the view type based on the folder:

- Inbox → `inboxViewType`
- Drafts → `draftViewType`
- Sent Items → `sentViewType`
- Other folders → `viewType`

This ensures each folder can maintain its own Read/Unread/All state.

---

## Paging and Continuation Tokens

Inbox supports paging using a continuation token:

- Token is stored per folder cache pack:
  - `inboxMails.nextToken`
  - `sentMails.nextToken`
  - `draftMails.nextToken`

On scroll:

- If no token exists, scrolling stops (no extra fetch)
- If token exists, the next page is fetched and merged into existing grouped results
- Duplicates are avoided by checking email `id`

---

## Sorting and Grouping

Inbox grouping is done by `sortEmailsOptimized(...)`.

### Supported Sort Modes

- `date` (default grouping):
  - Today
  - Yesterday
  - This Week
  - Last Week
  - This Month
  - Last Month
  - Month names (earlier months)
  - Year buckets (older mail)
- `category`
- `importance`
- `flagStatus`
- `from`
- `subject` (fallback grouping)

Within each group, items are sorted by date according to `sortOrder`.

---

## Refresh Behavior

Inbox refreshes using multiple strategies.

### Inbox Polling (Every 30 Seconds)

When the current folder is Inbox:

- `setInterval` triggers every 30 seconds
- Calls `EmailService.refreshInbox(0, 10, folderId)`
- New messages are merged into the current grouped list
- Folder counters are updated for Inbox if new mails arrive

### Refresh on Focus / Online

When the user returns to the tab or connectivity changes:

- Triggers an auto refresh with throttling
- Calls `getItems(0, 100, true)` to reconcile state

---

## Backend API

### Endpoint
```http
GET /Inbox
Authorization: Bearer <token>
```

### Authorization
```csharp
[Authorize(Roles = "ADMIN,MANAGER,STAFF")]
```

The backend derives the authenticated user from the token using `User.GetUserId()` and loads provider tokens from storage.

### Backend Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| start | int | 0 | Paging start index |
| length | int | 35 | Paging length |
| search | string | null | Search query |
| userId | string | "" | User id (backend also uses auth user id) |
| inboxType | string | "inbox" | Folder name/type |
| isImportant | bool | false | Important-only filter |
| viewType | string | "" | unread/read/all |
| isFlagged | bool? | null | Filter flagged |
| isStarred | bool? | null | Filter starred (mostly Gmail) |
| hasAttachment | bool? | null | Filter attachments |
| hasMentioned | bool? | null | Filter mentions |
| categoryType | string | null | Filter category |
| sortBy | string | "Date" | Sort field |
| nextPageToken | string? | null | Continuation token |
| startDate | string? | null | Date range start |
| endDate | string? | null | Date range end |

---

## Provider Routing (Backend)

The backend loads tokens and chooses provider:

### Load access token:
```csharp
ApplicationUserAccessTokens accessToken = await db.GetByUserId(User.GetUserId());
```

### If provider == Outlook:

Build Microsoft Graph client via `_graphSdkHelper.GetAuthenticatedClient(accessToken)`

Call:
```csharp
MicrosoftController.GetMessages(...)
```

### If provider == Gmail:

Instantiate `GoogleController`

Call:
```csharp
controller.GetMessageList(...)
```

### If unknown provider:

Return "Unsupported email provider"

---

## Outlook Provider Path

When provider is Outlook:

- Uses Microsoft Graph client

Calls:
```csharp
MicrosoftController.GetMessages(
  client,
  start,
  length,
  search,
  userId,
  inboxType,
  isImportant,
  viewType,
  isFlagged,
  hasAttachment,
  hasMentioned,
  categoryType,
  nextPageToken,
  sortBy,
  startDate,
  endDate
);
```

---

## Gmail Provider Path

When provider is Gmail:

Calls:
```csharp
controller.GetMessageList(
  User.GetUserId(),
  start,
  length,
  search,
  viewType,
  inboxType,
  isImportant,
  isStarred,
  hasAttachment,
  categoryType,
  nextPageToken,
  startDate,
  endDate
);
```

---

## Response Shape

The API responds with:
```csharp
ApiResponse<PagedData<EmailsListItem>>
```

The response typically contains:

- `items`: the list of email items
- `continuationToken`: optional token to fetch the next page

The frontend transforms the returned list into grouped results using `sortEmailsOptimized(...)`.

---

## Common UI State Updates

### Read/Unread State Update
- Server update happens via `EmailService.updateMessage(...)`
- UI updates via `updateEmails(..., EmailUpdateTypes.readUnread)`
- If current view is filtered (e.g., unread), items may be removed when the state no longer matches.

### Delete/Archive Update
- Server update happens via `EmailService.moveFolder(...)`
- UI removes item from grouped caches
- Triggers auto refresh to reconcile counts and server state

### Flag Update
- Server update happens via `EmailService.updateMessage(... Flag ...)`
- UI updates flag status and selected mail flag status (if open)

### Category Update
- Server update happens via `EmailService.updateMessage(... Category ...)`
- UI merges the category into local categories list and prevents duplicates
- Clear category resets categories array

---
