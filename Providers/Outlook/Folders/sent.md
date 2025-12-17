# Sent Folder

## Overview

The Sent folder contains all emails that have been successfully sent from the Mail module. It serves as a reliable record of outbound communication and provides full visibility into sent messages across both Gmail and Outlook accounts.

The Sent experience is fully provider-agnostic. Regardless of whether the underlying provider is Gmail or Outlook, sent messages are retrieved, displayed, and managed using the same unified Mail architecture and UI behavior.

All Sent operations are routed through the common Inbox API, with folder-specific behavior controlled through parameters.

---

## Unified Sent Architecture

- Sent uses the same common message retrieval API as Inbox and Drafts
- Folder identification determines Sent-specific behavior
- Provider routing is handled transparently based on user configuration
- Responses are normalized into a standard Mail response model
- The frontend remains completely unaware of the underlying provider

This ensures consistent Sent behavior across all supported email providers.

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

[Inbox Documentation](../API/InboxApi.md)

---

## Sent Folder Characteristics

- Contains successfully delivered outgoing emails
- Supports read and unread state (provider-dependent)
- Fully searchable and filterable
- Supports categories, flags, and importance
- Maintains independent paging and view state
- Cached separately from Inbox and Drafts

---

## Sent UI Experience

### Sent Screen

The Sent screen provides:

- Folder header with filter controls
- Unread / Read / All toggle buttons
- Debounced search input
- Grouped infinite-scrolling message list
- Right-click contextual menu actions
- Empty state messaging when no sent emails exist

Cached data is rendered immediately, followed by background synchronization.

---

## View Modes (Unread / Read / All)

Sent maintains its own view state:

- unread
- read
- all

When the view mode changes:

- The currently selected email is cleared
- Paging state is reset
- Messages are refetched using the common Inbox API
- Inbox and Drafts view states remain unaffected

Each folder remembers its own view preference.

---

## Search in Sent

- Search is scoped exclusively to the Sent folder
- Input is debounced to reduce unnecessary API calls
- Paging tokens are reset when a new search is performed
- Search resets automatically when switching folders

This ensures fast, isolated, and predictable search behavior.

---

## Grouping and Sorting

Sent supports the same sorting and grouping logic as other folders.

### Supported Sort Modes

- date (default)
  - Today
  - Yesterday
  - This Week
  - Last Week
  - This Month
  - Older month and year buckets
- category
- importance
- flagStatus
- from
- subject

Grouping and sorting are applied client-side to maintain consistent behavior across providers.

---

## Paging and Infinite Scroll

Sent uses continuation-token–based paging.

- Each response may include a continuation token
- Tokens are stored per Sent folder cache
- Infinite scrolling stops automatically when no token exists
- Newly fetched items are merged without duplication

Paging is fully isolated from Inbox and Drafts.

---

## Context Menu Actions (Sent)

Right-clicking a sent email opens a contextual menu with the following actions.

### Archive

- Archives the sent message (provider-mapped behavior)
- Removes the message from the Sent list
- Updates the local cache immediately

---

### Delete

- Moves the email to Deleted Items (or provider equivalent)
- Removes the message from Sent immediately
- Updates folder counters
- Clears the selected email if open

---

### Mark Read / Unread

- Updates the read state locally
- Automatically removes the message if it no longer matches the active filter

---

### Flag

Supports the following flag options:

- Today
- Tomorrow
- This Week
- Next Week
- Mark Complete
- Clear Flag

Flag changes are reflected immediately in both list and detail views.

---

### Categories

- Assign one or more categories
- Clear assigned categories
- Category updates are merged safely and deduplicated

---

## Local Cache and Performance

### Sent Cache

Sent maintains a dedicated cache:

- Cached messages render instantly when switching folders
- UI does not wait for network responses
- A background refresh reconciles the state with the server

This provides fast navigation without sacrificing accuracy.

---

## Refresh and Synchronization

Sent is refreshed automatically when:

- Filters or view modes change
- Search parameters update
- Mutations occur (delete, archive, flag, category)
- The folder is revisited after navigation

All refresh actions are throttled to prevent excessive API usage.

---

## Empty State Handling

When no sent emails are available:

- A friendly empty state is displayed
- Users are encouraged to adjust filters or search criteria
- UI remains stable and responsive

---

## Security and Reliability

- All Sent actions are authenticated using Bearer tokens
- Provider OAuth tokens are handled securely
- Provider-specific permissions are respected
- Errors are surfaced cleanly with user feedback

---

## Integration with the Mail Module

When the Sent folder is active:

- Requests flow through the same common Inbox API
- Folder parameters identify Sent-specific behavior
- Provider routing happens automatically
- Responses are normalized into the Mail module format
- UI behavior matches Inbox and Drafts exactly

No provider-specific UI logic is required.

---

## Key Takeaways

- Sent uses one common API, not a separate endpoint
- Behavior is folder-driven, not provider-driven
- Gmail and Outlook Sent folders behave identically
- Paging and caching are isolated per folder
- The architecture is scalable and maintainable

---

## Summary

The Sent folder provides a clear and reliable view of all outgoing communication within the Mail module. By leveraging a unified API, folder-driven behavior, intelligent caching, and consistent UI patterns, Sent ensures a predictable and high-performance experience regardless of whether the underlying email provider is Gmail or Outlook.
