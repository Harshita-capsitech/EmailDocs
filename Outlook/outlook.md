# Outlook Provider (Microsoft Email Integration)

## Overview

The Outlook provider enables full integration with Microsoft email services, including **Outlook.com** and **Microsoft 365** accounts. When your account is configured to use Outlook, all mail operations are powered by **Microsoft Graph APIs**, while maintaining the same unified experience provided by the Mail module.

The system automatically routes email requests to the Outlook provider based on your account configuration, ensuring seamless access to Outlook features without changing how you interact with the application.

---

## Microsoft API–Based Integration

- Outlook integration is implemented using **Microsoft Graph APIs**
- OAuth 2.0 authentication via Microsoft Identity Platform
- All email, folder, attachment, and action operations are executed through Microsoft-provided endpoints
- Secure access token storage with automatic refresh handling
- Encrypted communication with Microsoft services

---

## Supported Accounts

- Outlook.com
- Microsoft 365 (Exchange Online)
- Organization-managed Exchange accounts (where permitted)

---
# Mail Module – Frontend Architecture Highlights

## Unified State Hub
**MailContext** serves as the single source of truth, managing:
- Folder states  
- Email lists  
- Selections  
- Background polling  

All of this is exposed through the `useMail()` hook, ensuring predictable and centralized state management.

---

## Provider-Agnostic Design
The frontend is fully decoupled from specific email providers such as **Gmail** and **Outlook** by:
- Using a **normalized email data structure**
- Routing all requests through a central **EmailService**

This guarantees consistent behavior regardless of the underlying provider.

---

## Efficient Caching
Each mail folder maintains its own:
- Cache
- Paging tokens
- View parameters  

This design enables:
- Fast folder navigation
- Reduced API calls
- Better scalability for large mailboxes

---

## Optimized Client-Side Logic
To ensure a highly responsive UI and consistent behavior across providers:
- Sorting
- Grouping (by date, category, etc.)
- Filtering  

are handled entirely on the client side.

---

## Robust Mutation Handling
Email actions such as:
- Archiving
- Flagging
- Marking read/unread  

use **cache-safe updates** and **debounced refreshes**, keeping the UI synchronized with the server without introducing performance bottlenecks.

## Core Outlook Features

### Inbox and Folder Management

- Standard Outlook folders: Inbox, Sent, Drafts, Deleted Items, Junk
- Support for custom user-created folders
- Nested folder structures
- Automatic mapping to the Mail module’s unified folder model


> 📘 **Detailed Information**  
> For in-depth details about inbox behavior, folder synchronization, and provider-specific handling,  
> refer to: [Inbox & Folder Management](./Folders/inbox.md)

---

### Folder Retrieval Architecture

- A **separate Folder API** is used to retrieve all mail folders
- Each folder has a **unique `folderId`** provided by Microsoft
- Folder metadata includes:
  - Folder name
  - Unique folder ID
  - Parent folder reference
- Supports default, custom, and nested folders

---

### Folder-Based Inbox API

- A **single message retrieval API** is used for all folders
- This API internally targets the **Inbox endpoint**
- Messages are fetched by passing the corresponding **`folderId`**
- Enables reuse of one API for all folder-based message retrieval

---

### Message List and Reading Experience

- Optimized message listing with read/unread state
- Sender, subject, preview text, and timestamp display
- Conversation-based message grouping
- Smooth rendering of HTML and rich-content emails

---

### Compose, Reply, and Forward

- Compose new emails using Microsoft Graph APIs
- Reply, reply all, and forward messages
- Rich text formatting support
- Automatic draft saving
- Reliable message delivery through Microsoft infrastructure

---

### Attachments

- Upload and send file attachments
- Download attachments from received emails
- Clear attachment metadata (file name, size, type)
- Secure handling of attachments via Microsoft APIs

---

## Outlook-Specific Capabilities

### Focused Inbox

- Support for Outlook’s Focused Inbox model
- Separation of Focused and Other messages
- Focused and non-focused messages mapped cleanly in the UI

---

### Categories

- Outlook category support
- Color-coded message categorization
- Categories mapped to the unified tagging system where applicable

---

### Search

- Microsoft Graph–powered search
- Fast retrieval across folders
- Search by sender, subject, keywords, and date
- Optimized handling of large mailboxes

---

### Sorting and Filtering

- Sort by received date, sender, or subject
- Filter unread or flagged messages
- Provider-aware filtering logic applied automatically

---

## Real-Time Updates (Polling)

### Refresh API

- A **dedicated Refresh API** is implemented for inbox updates
- The API is triggered **every 3 seconds (polling mechanism)**
- Each refresh call includes:
  - Current `folderId`
  - User context

### Behavior

- Retrieves newly arrived or updated messages
- Updates the message list for the active folder
- Enables near real-time synchronization with Outlook without manual refresh

---

## Productivity and Actions

- Mark messages as read or unread
- Move messages between folders
- Delete and restore emails
- Flag messages for follow-up
- Bulk actions for faster inbox management

---

## Security and Compliance

- OAuth-based authentication with Microsoft
- Secure token storage and refresh handling
- Encrypted communication with Microsoft Graph APIs
- Compliance with Microsoft 365 security standards
- Tenant-level isolation for organizational accounts

---

## Performance and Reliability

- Efficient pagination for large folders
- Optimized API usage to reduce latency
- Background token refresh to avoid interruptions
- Graceful handling of Microsoft API throttling

---

## Integration with the Mail Module

When Outlook is your configured provider:

- All requests are routed through the unified `emails` controller
- The Outlook provider is selected automatically using your `userId`
- Folder IDs drive message retrieval
- Responses are normalized to match the Mail module’s standard format
- The UI remains consistent with other providers (e.g., Gmail)

You do not need to change how you use the application when switching to or from Outlook.

---

## Limitations and Considerations

- Some Outlook features may depend on account type or tenant policies
- Shared mailbox access requires appropriate permissions
- Feature availability may vary between Outlook.com and Microsoft 365 accounts
- Polling frequency may be adjusted to comply with Microsoft API limits

---

## Summary

The Outlook provider delivers a secure, enterprise-ready email experience powered by **Microsoft Graph APIs**. By combining a dedicated folder retrieval API, a unified inbox API driven by folder IDs, and a 3-second polling refresh mechanism, the Mail module ensures real-time updates, scalability, and a consistent user experience.
