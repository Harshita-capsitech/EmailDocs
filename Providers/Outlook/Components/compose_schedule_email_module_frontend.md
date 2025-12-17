# Compose & Schedule Email Module

## Overview
This module implements the **email compose experience** in the frontend, supporting **draft creation, auto-save, attachments, importance levels, tone analysis, and scheduled sending**. It integrates tightly with backend email APIs (Outlook / Gmail) via `EmailService`.

---

## Key Features

- Create and manage **email drafts** automatically
- Send emails immediately or **schedule for later delivery**
- Support **To / Cc / Bcc** recipients with async user search
- Handle **file attachments** (small + chunked uploads)
- Set **High / Low importance**
- Perform **tone analysis** while typing
- Full-page compose mode
- Draft discard confirmation

---

## Primary Components

### 1. `ComposeMessage`
Main compose UI component responsible for drafting, editing, sending, and scheduling emails.

#### Responsibilities
- Initialize draft on load
- Auto-save draft every 30 seconds on changes
- Manage recipients, subject, body, attachments
- Trigger send or schedule flows

#### Props
```ts
ComposeMessageProps {
  onClose: () => void;
  draftId?: string;
  clientDataRequestId?: string;
  onDismiss?: () => void;
}
```

---

## Draft Lifecycle

### Draft Initialization
```ts
EmailService.saveAsDraft({
  requestType: EmailRequestType.Compose,
  importance: Importance.Normal,
});
```
- Draft is created when compose opens
- Stored in state via `useMail()`

### Auto Save Draft
- Triggered every **30 seconds** when form changes
- Updates draft content and attachment metadata

```ts
EmailService.saveAsDraft({ ...values, id: draftId });
```

---

## Send Email

### API Used
```ts
POST /email/send
```

### Trigger
- User clicks **Send** button
- Draft is saved first, then sent

```ts
EmailService.sendMessage({ ...values, id: draftId });
```

### Success Flow
- Draft count updated
- Draft cleared from state
- Toast: `Mail sent`

---

## Schedule Email

### UI Component
`ScheduleDateTimeView`

### Supported Options
- 1 hour later
- Today (Noon / Evening)
- Tomorrow morning
- Later this week / Monday
- Custom date & time

### API Used
```ts
POST /email/schedule
```

### Payload Example
```json
{
  "id": "draftId",
  "requestType": 4,
  "toRecipients": ["user@email.com"],
  "ccRecipients": [],
  "bccRecipients": [],
  "subject": "Subject",
  "body": "HTML body",
  "attachments": []
}
```

---

## Attachments Handling

### Supported Formats
- Documents: pdf, doc, docx, xls, ppt
- Media: images, audio, video
- Archives: zip, rar, gz

### Upload Strategy
| File Size | Method |
|---------|--------|
| ≤ 2MB | Single upload |
| > 2MB | Chunked upload |

### APIs Used
```ts
POST /email/attachments
POST /email/attachments/chunked
DELETE /email/attachments/{id}
```

---

## Tone Analysis

- Triggered while typing email body
- Uses `useAnalyzeTone()` hook
- Displays detected tones as badges

```ts
debouncedAnalyzeTone(bodyText);
```

---

## Importance Handling

- High Importance
- Low Importance

```ts
values.importance = Importance.High | Importance.Low
```

Visual feedback provided via icon state.

---

## Validation Rules

- At least **one To recipient** required
- Subject is mandatory
- Email format validation for recipients

Implemented using **Yup + Formik**.

---

## Error Handling

- Draft save failure
- Attachment upload failure
- Schedule validation errors
- API errors surfaced via toast messages

---

| Feature | API | Documentation |
|---------|-----|---------------|
| Save Draft | `POST /email/draft` | [Draft API](../API/DraftApi.md) |
| Send Email | `POST /email/send` | [Send API](../API/PostMessage.md) |
| Schedule Email | `POST /email/schedule` | [](./operations/schedule.md) |
| Upload Attachment | `POST /email/attachments` | [Attachments](../API/UploadAttachments/PostAttachments.md) |
| Chunk Upload | `POST /email/attachments/chunked` | [Chunked Upload](../API/UploadAttachments/uploadAttachmentInChunks.md) |
| Delete Attachment | `DELETE /email/attachments/{id}` | [Attachment Management](../API/deleteAttachment.md) |
---

## Notes

- Draft ID is mandatory for attachments
- Gmail & Outlook handled transparently by backend
- Designed to match Outlook-like compose experience

---

## Status
**Production-ready frontend compose module**

