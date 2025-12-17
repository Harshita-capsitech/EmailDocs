# Drafts Folder

## Overview

The Drafts folder represents all emails that have been created but not yet sent. It provides a reliable, auto-saved workspace where users can safely compose, edit, and manage unfinished messages.

Drafts follows the same unified, provider-agnostic architecture as Inbox and Sent. Whether the underlying provider is Gmail or Outlook, the Drafts experience remains consistent, predictable, and fully integrated into the Mail module.

All Drafts operations are routed through the common Inbox API, with folder-specific behavior controlled by parameters.

## Unified Drafts Architecture

- Drafts uses the same common message retrieval API as all other folders
- Folder-specific behavior is determined using the Drafts folder identifier
- Provider detection (Gmail or Outlook) happens transparently
- Responses are normalized into a unified format
- The frontend remains completely provider-agnostic

This ensures Drafts behaves identically across email providers.

## Drafts Folder Characteristics

- Contains unsent email messages
- Supports read and unread states
- Fully searchable and filterable
- Supports categories, flags, and importance
- Maintains its own paging and view state
- Cached independently from Inbox and Sent

## Drafts UI Experience

### Drafts Screen

The Drafts screen provides:

- Folder header with filter controls
- Unread / Read / All toggle buttons
- Search input with debounce
- Grouped infinite-scrolling list
- Right-click contextual actions
- Empty state messaging when no drafts exist

The UI updates immediately using cached data, followed by background reconciliation.

### View Modes (Unread / Read / All)

Drafts maintains its own view toggle state:

- Unread
- Read
- All

Changing the view:

- Clears the currently selected draft
- Resets paging state
- Triggers a refetch using the common Inbox API
- Does not affect Inbox or Sent views

Each folder remembers its own view preference.

### Search in Drafts

- Search is scoped strictly to Drafts
- Input is debounced for performance
- Paging tokens are reset on new search
- Search resets automatically when switching folders

This ensures fast and isolated search behavior.

## Grouping and Sorting

Drafts supports the same grouping and sorting capabilities as Inbox:

### Supported Sort Modes

**Date (default)**
- Today
- Yesterday
- This Week
- Last Week
- This Month
- Older month and year buckets

**Category**

**Importance**

**Flag status**

**From / To**

**Subject**

Grouping and sorting are performed client-side to ensure identical behavior across providers.

## Paging and Infinite Scroll

Drafts uses continuation-token–based paging.

- Each fetch returns a continuation token
- Tokens are stored per-folder
- Infinite scrolling stops automatically when no token exists
- Newly fetched items are merged without duplication

Drafts paging is fully isolated from Inbox and Sent paging.

## Context Menu Actions (Drafts)

Right-clicking a draft opens a contextual menu with the following actions.

### Archive

- Removes the draft from Drafts
- Updates local cache immediately
- Updates folder count
- Clears selected draft if open

### Delete

- Permanently deletes the draft
- Removes it from all Drafts caches
- Updates navigation counters
- Triggers background reconciliation

### Mark Read / Unread

- Toggles read state
- Automatically removes the item if it no longer matches the active view filter

### Flag

Supports the following flag presets:

- Today
- Tomorrow
- This Week
- Next Week
- Mark Complete
- Clear Flag

Flag updates are reflected immediately in both list and detail views.

### Categories

- Assign one or more categories
- Clear assigned categories
- Categories are merged safely without duplication

## Local Cache and Performance

### Drafts Cache

Drafts maintains a dedicated cache:

- Cached data is reused instantly when switching folders
- UI renders immediately without waiting for the server
- A background refresh reconciles state shortly after

This results in fast navigation with no stale data risk.

### Refresh and Synchronization

Drafts is refreshed automatically when:

- Filters or view modes change
- Search parameters update
- Mutations occur (delete, archive, flag, category)
- Folder is revisited after navigation

All refresh actions are throttled to prevent excessive API usage.

### Empty State Handling

When Drafts contains no messages:

- A friendly empty state is displayed
- Users are prompted to adjust filters or search terms
- UI remains responsive and predictable

## Security and Reliability

- All Drafts actions are authenticated via Bearer token
- Provider access tokens are managed securely
- Provider-specific rules and permissions are respected
- Errors are handled gracefully with user feedback

## Integration with the Mail Module

When Drafts is active:

- Requests flow through the same common Inbox API
- Folder-specific parameters identify Drafts
- Provider routing happens automatically
- Responses are normalized into the standard Mail format
- UI behavior matches Inbox and Sent exactly
- No special handling is required at the UI level

## Key Takeaways

- Drafts uses one common API, not a separate endpoint
- Behavior is folder-driven, not provider-driven
- Gmail and Outlook Drafts behave identically
- Caching and paging are isolated per folder
- The architecture is scalable and maintainable

## Summary

The Drafts folder delivers a reliable, high-performance drafting experience built on the Mail module's unified architecture. By combining provider-agnostic APIs, folder-specific state management, intelligent caching, and consistent UI behavior, Drafts ensures users never lose work and can manage unfinished emails with confidence—regardless of whether the underlying provider is Gmail or Outlook.