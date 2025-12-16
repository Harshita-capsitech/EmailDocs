# Mail Context & State Management

This document explains the **MailContext**, **MailProvider**, and related logic used to manage email state across the application.  
The design is **provider-agnostic** and supports both **Outlook and Gmail** through a unified frontend state model.

---

## Purpose of MailContext

`MailContext` acts as the **single source of truth** for all mail-related state across the application.

It manages:

- Current folder (Inbox, Sent, Drafts, etc.)
- Email lists and per-folder caches
- Selected email (detail view)
- Filters, sorting, and paging
- Background refresh and inbox polling
- Email mutations such as read/unread, flag, delete, archive, category, and star

All inbox screens, detail views, and email actions consume this state using the `useMail()` hook.

---

## Core Concepts

### Provider-Agnostic Design

The frontend does **not** depend on whether the email provider is Gmail or Outlook.

- Provider detection happens on the backend
- Frontend always communicates through `EmailService`
- API responses are normalized into a common structure
- Sorting, grouping, and filtering are handled locally

This keeps UI logic simple, predictable, and consistent across providers.

---

## Mail State Structure

### Global Mail State (`MailState`)

selectedMail
isLoading
currentFolder
params
draftMessage
draftMessageInitial


This state controls:

- Which email is currently open
- Which folder is active
- Applied filters and sorting
- Loading indicators for list, detail, refresh, or global operations

---

### Collection State (`MailCollectionState`)

folders
emails
inboxMails
sentMails
draftMails
importantMails
categories


This state stores:

- Cached emails per folder
- Folder metadata and unread/message counters
- Category definitions used for tagging

Each folder maintains its **own cache and paging token**, enabling fast navigation and reduced API calls.

---

## Email List Parameters

The `IEmailListParamsState` object controls how emails are fetched, filtered, and displayed.

| Field | Purpose |
|-----|--------|
| search | Text-based search |
| viewType | Default view (read / unread / all) |
| inboxViewType | Inbox-specific view |
| sentViewType | Sent-specific view |
| draftViewType | Draft-specific view |
| isFlagged | Flag filter |
| isStarred | Star filter |
| isImportant | Importance filter |
| hasAttachment | Attachment filter |
| isMentioned | Mention filter |
| categoryType | Category filter |
| sortBy | Sorting strategy |
| sortOrder | Ascending or descending |
| nextPageToken | Paging token |

Each folder can **remember its own parameters**, preserving user context when switching folders.

---

## Folder Handling

### Folder Initialization

- Folders are fetched using `EmailService.getFolders()`
- Icons are resolved using a centralized `iconMapping`
- Inbox is auto-selected on initial load
- Folder counters are displayed in navigation

---

### Folder Switching (`setFolder`)

When a user switches folders:

1. Current folder parameters are saved
2. Selected mail and drafts are cleared
3. Cached data is reused if available
4. Server reconciliation happens shortly after

This ensures:

- No unnecessary reloads
- Fast UI response
- Consistent server-backed data

---

## Fetching Emails

### Main Fetch Function

getItems(start, length, reset, onScroll, folderName)


Key behaviors:

- Uses `AbortController` to cancel stale requests
- Applies folder-specific paging tokens
- Uses folder-specific view types
- Merges results without duplicates
- Updates both the global list and folder cache

---

### Folder-Specific View Logic

| Folder | View Param Used |
|------|---------------|
| Inbox | inboxViewType |
| Sent | sentViewType |
| Drafts | draftViewType |
| Others | viewType |

This allows each folder to independently maintain its read/unread state.

---

## Infinite Scrolling

- Triggered via `onScroll`
- Throttled to prevent excessive API calls
- Automatically stops when no continuation token exists
- Safely merges new items into existing grouped data

---

## Sorting & Grouping

Sorting and grouping are handled by `sortEmailsOptimized()`.

### Supported Grouping Modes

- Date  
  Today, Yesterday, This Week, Last Week, This Month, Last Month, Year
- Category
- Importance
- Flag Status
- From
- To
- Subject

All grouping is performed **client-side**, ensuring identical behavior across Gmail and Outlook.

---

## Local Filtering

Filters are applied locally using `applyLocalFilters()`:

- Flagged
- Starred
- Important
- Attachments
- Mentions
- Categories

This minimizes API calls and keeps interactions responsive.

---

## Email Mutations

### Supported Mutations

| Action | Update Type |
|------|------------|
| Read / Unread | readUnread |
| Flag | flag |
| Category | category |
| Delete | delete |
| Archive | archive |
| Star | star |

Each mutation:

1. Calls the backend API
2. Updates local grouped lists
3. Updates folder-specific caches
4. Triggers a soft refresh

---

### Cache-Safe Updates

Updates rely on:

- Group-level transformations
- ID-based replacements
- Duplicate prevention
- Re-grouping after mutation

This keeps UI state consistent even during rapid user actions.

---

## Auto Refresh Strategy

### Inbox Polling

When Inbox is active:

- Polls every **30 seconds**
- Fetches new messages
- Prepends new emails to the list
- Updates folder counters automatically

---

### Focus & Network Refresh

Refresh is triggered when:

- Browser tab becomes visible
- Network reconnects
- Mutations occur

All refresh actions are **debounced** to avoid request storms.

---

## Search Behavior

- Search updates `params.search`
- Input is debounced
- Paging token is reset
- Folder context is preserved

---

## Categories

Categories are:

- Loaded once during initialization
- Styled using `categoryColors`
- Applied per email
- Deduplicated locally

Clearing a category resets the email’s category list.

---

## Error Handling

- All API errors surface via `AOToast.error`
- Abort errors are silently ignored
- UI state always resolves cleanly

---

## Why This Architecture Works

- **Provider-agnostic**: Gmail and Outlook behave identically in the UI  
- **Fast**: Aggressive caching and local filtering  
- **Safe**: Abortable requests and debounced refresh  
- **Scalable**: Easy to add new folders or providers  
- **Predictable**: One-way data flow with controlled mutations  

---


