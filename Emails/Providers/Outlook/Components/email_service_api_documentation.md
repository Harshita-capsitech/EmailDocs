# EmailService – Frontend API Layer Documentation

## Overview
`EmailService` is a centralized frontend service responsible for **email operations**, **conversation handling**, **folder management**, **attachments (including chunk uploads)**, and **AI-assisted email features**. It communicates with the backend Email APIs and AI services using a standardized `ApiUtility` wrapper.

This service supports **Inbox operations, Threads, Drafts, Scheduling, Attachments, Preferences, and AI-powered tools**.

---

## Base Configuration

### Email API
- **Base URL**: `REACT_APP_EMAIL_API_URL`
- **Route Prefix**: `/emails`

### AI Email API
- **Base URL**: `REACT_APP_EMAIL_AI_API_URL/api/convomail`

---

## Core Models & Enums

### EmailRequestType
| Value | Description |
|-----|------------|
| Reply | Reply to sender |
| ReplyAll | Reply to all recipients |
| Forward | Forward message |
| Compose | New email |
| Draft | Draft email |

### EmailUpdateType
| Type | Description |
|-----|------------|
| ReadUnRead | Toggle read state |
| Flag | Update follow-up flag |
| Category | Add category |
| RemoveSingleCategory | Remove category |
| Star | Toggle star |

### Importance
- Low
- Normal
- High

### FollowUpFlagStatus
- NotFlagged
- Complete
- Flagged

---

## Inbox & Message APIs

### Get Inbox Messages
```ts
getInboxMessages(start, length, search?, userId?, inboxType?, isImportant?, viewType?, isFlagged?, isStarred?, hasAttachment?, hasMentioned?, categoryType?, sortBy?, nextPageToken?, startDate?, endDate?)
```
**Purpose**: Fetch paginated inbox messages with advanced filters.

---

### Refresh Inbox
```ts
refreshInbox(start, length, inboxType?)
```
**Purpose**: Fetch newly received emails since last poll.

---

### Get Single Message
```ts
getMessage(messageId)
```
**Purpose**: Retrieve a full email message.

---

### Delete Message
```ts
deleteMessage(messageId)
```

---

## Conversations & Threads

### Get Conversation Messages
```ts
getConversationMessages(conversationId)
```
**Purpose**: Retrieve all messages belonging to a conversation thread.

---

### Get Reply Count
```ts
getReplyCount(messageId)
```

---

## Sending & Drafts

### Send / Reply / Forward
```ts
sendMessage(param)
replyMessage(messageId, param)
replyAllMessage(messageId, param)
forwardMessage(messageId, param)
```

---

### Save Draft
```ts
saveAsDraft(param)
saveAsDraftInitial(param)
```

---

### Scheduled Emails
```ts
scheduleMessage(datetime, model)
getScheduleList(start, length, ...)
updateSchedule(id, dateTime?, moveToDraft?)
```

---

## Folder Management

### Get Folders
```ts
getFolders()
```

### Create Folder
```ts
createFolder(param)
```

### Move Folder
```ts
moveFolder(id, dest)
```

---

## Categories & Preferences

### Get Categories
```ts
getCategories()
```

### Update Preferences
```ts
updatePref(userId, param)
```

---

## Attachments

### Upload Attachment (Direct)
```ts
postAttachment(formData, messageId, onUploadProgress?, signal?)
```

### Chunk Upload (Large Files)
```ts
createChunkUploadSession(messageId, file)
uploadAttachmentInChunks(file, messageId, onProgress?, options?)
```

**Flow**:
1. Create upload session
2. Upload file in chunks
3. Poll status if finalization pending
4. Resolve attachment ID

---

### Download Attachments
```ts
downloadAttachment(messageId, attachmentId)
downloadAllAttachments(messageId)
```

---

## Draft & Scheduled Attachments
```ts
draftFileDownload(draftId, fileId)
scheduleFileDownload(mailId, fileId)
```

---

## AI-Powered Email Features

### Translate
```ts
translateText(text[], tgtLang?)
```

### Summarize
```ts
generateSummary(text, isLong, feedbackIndex?, customFeedback?)
```

### Tone Analysis
```ts
analyzeTone(text)
```

### Auto Draft / Reply
```ts
generateAutoResponse(text)
generateReply(text)
generateAutoComplete(text)
```

---

## Initial Message APIs

### Initial Compose Context
```ts
getInitialMessage(params)
```

### Bookkeeping Client Initial Message
```ts
getBkClientInitialMessage(clientId, params)
```

---

## Security & Auth
- Uses **OIDC-based Axios config**
- Fraud prevention headers added where required
- Credentials passed via cookies

---

## Error Handling
- Centralized via `ApiUtility`
- Backend errors propagated consistently
- Chunk uploads handle retry & polling logic

---

## Summary
This service acts as the **single source of truth** for all email-related frontend interactions, ensuring:
- Clean separation of concerns
- Consistent A