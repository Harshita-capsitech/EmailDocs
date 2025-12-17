# Inbox

## Overview

The Inbox feature provides a single, consistent inbox experience while supporting **both Gmail and Outlook** behind the scenes. The UI and API remain **provider-agnostic**, while the system internally routes each request to the correct provider implementation based on the authenticated user.

This document explains how the Inbox works end-to-end **from Frontend → Common Inbox API → Provider routing**, covering paging, filters, sorting, search, and refresh behavior.

> 🔗 Related docs:  
> - [Mail Module Overview](../../../Architecture/Overview.md)  
> - [Outlook Provider](../outlook.md)

---

## End-to-End Flow

### Frontend → Common Inbox API → Provider

1. The Inbox screen loads emails using `useMail()` (MailContext).
> 👉 **[Click here to view Mail Context](../components/mailContext.md)**
2. `MailContext.getItems()` calls `EmailService.getInboxMessages(...)`.
3. All folder-based email requests (Inbox, Sent, Drafts, etc.) are routed through a **single common API**:
/api/inboxapi|
4. The API identifies the authenticated user from the Bearer token.
5. Based on the user configuration, the API determines the email provider.
6. The request is internally routed:
- Outlook → Microsoft provider logic
- Gmail → Google provider logic
7. The API always returns a **unified response model**, which the frontend groups and renders.

Conceptual routing:


User Request → /api/inboxapi → Provider Flag Check
├─→ Outlook Provider
└─→ Gmail Provider

---

## Common API for All Folders

A **single API** (`/api/inboxapi`) is used for **all mail folders**, including:

- Inbox
- Sent Items
- Drafts
- Important
- Custom folders

Folder behavior is controlled using request parameters (such as folder id/type), not separate APIs.

This ensures:

- Consistent behavior across folders
- Shared paging and filtering logic
- Easier maintenance and extensibility

---

## Frontend Implementation

### Inbox UI Component

The Inbox UI is implemented in `NewInboxEmails` and includes:

- Header and filter menu
- Toggle buttons: **Unread / Read / All**
- Debounced search input
- Grouped infinite list (`ControlledGroupList`)
- Context menu actions (right-click)

---

## View Toggles (Unread / Read / All)

Inbox view toggles are controlled through `listParams.inboxViewType`:

- `unread`
- `read`
- `all`

When the view changes:

- `selectedMail` is cleared
- Paging state (`nextPageToken`) is reset
- Emails are refetched using the common Inbox API

---

## Search

Search is implemented using a debounced input (500ms).

Behavior:

- User types in the search box
- `updateParams({ search, nextPageToken: undefined })` is called
- The next fetch uses the updated search term
- Search is reset automatically when switching folders

---

## Grouped Infinite List

Inbox uses a grouped list renderer (`ControlledGroupList`).

- Emails are grouped based on the active sort mode
- Default grouping is by **date buckets** (Today, Yesterday, etc.)

Infinite scroll behavior:

- `onLoadMore` calculates the offset
- Calls `onScroll(start, PAGE, 'inbox')`
- Additional data is fetched via continuation tokens
- Results are merged without duplicates

---

## Context Menu Actions

Right-clicking an email opens a contextual menu with the following actions.

### Archive

- Moves the email out of Inbox
- Updates local cache using `EmailUpdateTypes.archive`

### Delete

- Moves the email to Deleted Items
- Updates local cache and removes it from the list

### Mark Read / Unread

- Updates read state
- If the current view is filtered (Unread/Read), items may disappear automatically

### Flag

Supports:

- Today
- Tomorrow
- This Week
- Next Week
- Mark Complete
- Clear Flag

Updates flag state locally using `EmailUpdateTypes.flag`.

### Categories

- Assign category
- Clear category

Local state merges categories safely and prevents duplicates.

---

## Folder-Specific View Logic

Although the API is common, the frontend applies **folder-specific view logic**:

- Inbox → `inboxViewType`
- Drafts → `draftViewType`
- Sent Items → `sentViewType`
- Other folders → `viewType`

This allows each folder to maintain its own Read / Unread / All state.

---

## Paging and Continuation Tokens

Paging is handled using continuation tokens returned by the common Inbox API.

Tokens are stored per folder cache:

- `inboxMails.nextToken`
- `sentMails.nextToken`
- `draftMails.nextToken`

Behavior:

- If no token exists, scrolling stops
- If a token exists, the next page is fetched
- Items are merged without duplication using email `id`

---

## Sorting and Grouping

Sorting and grouping are handled on the frontend using `sortEmailsOptimized(...)`.

### Supported Sort Modes

- `date` (default)
  - Today
  - Yesterday
  - This Week
  - Last Week
  - This Month
  - Last Month
  - Month names
  - Year buckets
- `category`
- `importance`
- `flagStatus`
- `from`
- `subject`

Within each group, emails are sorted by date based on `sortOrder`.

---

## Refresh Behavior

Inbox refresh uses multiple strategies to keep data up to date.

### Inbox Polling (Every 30 Seconds)

When Inbox is the active folder:

- Periodic polling fetches the latest emails
- New emails are merged at the top
- Folder counters are updated automatically

---

### Refresh on Focus / Connectivity Change

An automatic refresh is triggered when:

- The browser tab becomes visible
- Network connectivity changes
- Email mutations occur

All refresh actions are throttled to avoid excessive API calls.

---

## Common API Reference

All folder-based email operations use:


GET /api/inboxapi

Authorization: Bearer <token>

Key characteristics:

- Single API for all folders
- Provider-agnostic
- Folder behavior controlled by parameters
- Unified response format for Gmail and Outlook

---

## Response Shape

The common Inbox API returns a unified paged response containing:

- `items`: list of email items
- `continuationToken`: optional token for next page

The frontend transforms the returned items into grouped lists using `sortEmailsOptimized(...)`.

---

## Common UI State Updates

### Read / Unread Updates

- Local state updates immediately
- Filtered views auto-adjust if the item no longer matches

### Delete / Archive Updates

- Item is removed from local grouped caches
- Auto refresh reconciles counts and server state

### Flag Updates

- Flag state updates in list and detail view (if open)

### Category Updates

- Categories are merged locally
- Clearing a category resets the category list for that email

---

## Key Takeaways

👉 [One common API](../API/InboxApi.md) powers all folders
- Frontend logic is fully provider-agnostic
- Gmail and Outlook behave identically in the UI
- Paging, filtering, and grouping are consistent everywhere
- The architecture is scalable and easy to extend

This design ensures a predictable, high-performance inbox experience regardless of the underlying email provider.
