# Draft Edit Composer Module

## Overview

The **Draft Edit Composer module** powers the “edit draft” experience in the Emails UI. It allows users to open an existing draft, modify recipients/content/importance, manage attachments (single-request and chunked upload), and ultimately send the message.

This module solves several practical problems in a provider-based email system:

- Provides a **single draft editing experience** regardless of provider (Outlook now, Gmail later)
- Ensures draft changes are **validated** and **auto-saved**
- Supports **large attachment uploads** safely using chunk upload sessions
- Keeps UI state consistent across folders (Drafts/Inboxes/Sent) through the shared `MailContext`

### Key Responsibilities

- Initialize the draft form from `selectedMail`
- Validate recipients/subject/body via Formik + Yup
- Auto-save draft updates (every 30s when changes occur)
- Upload attachments with size-based strategy (≤4MB direct, >4MB chunked, max 30MB)
- Send draft → updates folder counters + removes item from cached lists

---

## Unified Entry Point

### `DraftEdit` Component

`DraftEdit` is the unified entry component for the draft-edit UI.

**Why a single entry point is used**
- Centralizes draft initialization and submission logic
- Keeps attachment handling consistent with the same UI and service calls
- Ensures all mutations go through the same state + validation path

**Supported operations**
- Update recipients (`To`, `Cc`, `Bcc`)
- Update subject/body
- Mark importance (High/Low)
- Auto-save draft
- Upload/Delete attachments
- Send draft
- Discard changes

---

## Input Models

### UI State Inputs (from Context)

| Property | Type | Source | Purpose |
|---|---|---|---|
| `selectedMail` | `OutlookSingleMailResponse \| null` | `MailContext` | Selected draft/message used to prefill form |
| `folders` | `INavLink[]` | `MailContext` | Folder list and counts (Draft count decremented after send) |
| `draftMessage` | `{ id: string; name: string } \| undefined` | `MailContext` | Tracks current draft message |
| `removeEmail` | `(id: string) => void` | `MailContext` | Removes message from lists/caches after send |

### Form Model (`EmailRequestModel`)

| Property | Type | Purpose |
|---|---|---|
| `id` | `string` | Draft/message id |
| `toRecipients` | `IIdName[]` | Primary recipients |
| `ccRecipients` | `IIdName[]` | CC recipients |
| `bccRecipients` | `IIdName[]` | BCC recipients |
| `subject` | `string` | Email subject |
| `body` | `string` | Editor HTML body |
| `content` | `string` | Content field (kept for compatibility) |
| `attachments` | `any[]` | Attachment list shown in UI |
| `requestType` | `EmailRequestType` | Set to `Draft` |
| `importance` | `Importance` | High/Normal/Low |

### Attachment Upload Inputs

| Property | Type | Source | Purpose |
|---|---|---|---|
| `file` | `File` | Drop zone | File selected for upload |
| `MAX_SIZE` | `number` | UI constant | 30MB hard limit |
| `CHUNK_THRESHOLD` | `number` | UI constant | 4MB switch threshold |

---

## Core Concepts / Normalization Logic

### Draft Initialization

When `selectedMail` changes, the form is rebuilt from the selected message:

- Filters out current user from `to/cc/bcc` recipient lists
- Populates editor via `_editor.current?.setData(val.body)`
- Copies server-side attachments into `uploadedFiles` and form state

### Validation Rules (Formik + Yup)

- `toRecipients` must contain at least one valid email object
- `subject` required
- `body` required
- Attachments are optional

### Auto-save Behavior

Draft auto-save is driven by the `isChange` flag:

- When the user changes any key field (recipients/subject/body/attachments), `isChange = true`
- A timer runs every **30 seconds** to call `EmailService.saveAsDraft(...)`
- On success, the UI shows “Draft saved at HH:MM AM/PM”
- After save completes, `isChange` is reset to `false`

### Upload Strategy Selection

Files are routed by size:

- `size <= 4MB` → direct upload via `EmailService.postAttachment(...)`
- `size > 4MB` → chunk workflow via `EmailService.uploadAttachmentInChunks(...)`
- `size > 30MB` → rejected in UI

---

## Base Object Construction

### Draft Form Values Builder

A normalized draft request model (`EmailRequestModel`) is built from `selectedMail` to ensure:
- UI always edits a consistent structure
- Provider-specific fields don’t leak into the editor
- Attachments have stable shape for list rendering + deletion

### Stable Temporary Upload ID

A `tempId` is created per upload attempt:
- Used to drive progress updates and UI state before the server returns a real attachment id

---

## Internal Helpers / Services

### `EmailService`

Used for:
- `saveAsDraft(...)` — auto-save and pre-send save
- `sendMessage(...)` — send draft
- `postAttachment(formData, draftId, onProgress, signal)` — direct upload
- `uploadAttachmentInChunks(file, draftId, onProgress, opts)` — chunk upload
- `getMessage(draftId)` — used to resolve attachment id after chunk upload
- `deleteAttachment(draftId, attachmentId)` — remove attachment on server

### `MailContext`

Used for:
- Folder count updates (`Drafts` count decremented after send)
- Removing the sent draft from cached lists via `removeEmail(...)`
- Resetting selection state (`selectedMail.set(null)`)

---

## Execution Flow by Action Type

### Edit Draft (Open + Prefill)

**Trigger conditions**
- User selects a draft from Drafts list

**Flow**
1. `selectedMail` updated in `MailContext`
2. `DraftEdit` `useEffect` rebuilds `EmailRequestModel`
3. Formik values set via `draftFormRef.current.setValues(val)`
4. Editor initialized via `_editor.current?.setData(val.body)`
5. Existing attachments loaded into UI

**Special considerations**
- Current user is filtered out of recipient arrays
- `importance` toggles (High/Low) are derived from the selected message

---

### Auto-save Draft

**Trigger conditions**
- Any user change sets `isChange = true`

**Flow**
1. `isChange` becomes `true`
2. Timer fires every 30s
3. Calls `EmailService.saveAsDraft({ ...val, Id: selectedMail.id, requestType: Draft })`
4. On success, updates `draftTime`
5. Resets `isChange = false`

**Special considerations**
- Only runs while `isChange` is true
- Uses a repeating interval but is cleared on cleanup

---

### Upload Attachment (≤ 4MB)

**Trigger conditions**
- File dropped with size `<= CHUNK_THRESHOLD`

**Flow**
1. Build `FormData` with `files`
2. Call `EmailService.postAttachment(formData, draftId, onProgress, signal)`
3. Map returned attachments into form model (`values.attachments`)
4. Mark UI upload as success

**Special considerations**
- Uses progress callback to keep UI responsive
- Avoids duplication by merging attachments into existing list

---

### Upload Attachment (> 4MB and ≤ 30MB)

**Trigger conditions**
- File dropped with size `> CHUNK_THRESHOLD`

**Flow**
1. Call `EmailService.uploadAttachmentInChunks(file, draftId, onProgress, abortController)`
2. After upload completes, UI attempts to resolve server attachment id:
   - Calls `EmailService.getMessage(draftId)` multiple times with backoff
   - Matches by name + size (fallback match by name + latest modified)
3. On success, adds resolved attachment to form values

**Special considerations**
- Server may take time to finalize attachment availability; hence the polling/backoff
- UI caps progress at 99% until completion is confirmed (prevents “fake 100%”)

---

### Delete Attachment

**Trigger conditions**
- User deletes an attachment from the UI list

**Flow**
1. Resolve `draftId`
2. Resolve `attachmentId` (direct from UI; fallback resolves via `getMessage`)
3. Call `EmailService.deleteAttachment(draftId, attachmentId)`
4. Remove from `values.attachments`

**Special considerations**
- Handles cases where attachment id is not immediately available (post-chunk)

---

### Send Draft

**Trigger conditions**
- User clicks **Send**

**Flow**
1. `onSubmit` begins; sets loading true
2. Calls `EmailService.saveAsDraft({ ...values, id: selectedMail.id })`
3. On success, calls `EmailService.sendMessage(values)`
4. On send success:
   - Decrement Drafts folder count in `MailContext.folders`
   - Clear `draftMessage`
   - `removeEmail(values.id)` to remove from UI caches
   - Clear selection via `selectedMail.set(null)`
5. Show toast “Mail sent successfully”

**Special considerations**
- Save-as-draft before send ensures server has the latest body/recipients/attachments
- Folder count mutation is localized and immediate for UI responsiveness

---

## Attachment / Asset Handling

### Upload Strategy
- ≤ 4MB: direct `multipart/form-data` upload
- > 4MB: chunk session upload with progress callback
- Max allowed: 30MB per file

### Retrieval Strategy
- Attachments are stored as part of the draft message; `getMessage(draftId)` returns the attachment list used for id resolution

### Sync Strategy
- UI maintains optimistic attachment list entries, then reconciles with server ids (especially for chunk uploads)

---

## Scheduling / Metadata Handling

### Auto-save Scheduling
- Interval: 30 seconds
- Condition: only active when `isChange === true`

### Validation Rules
- Uses Formik schema; prevents send without required fields

### Limitations
- Auto-save interval is fixed; very rapid edits may wait up to 30s unless user sends immediately (which triggers save)

---

## Error Handling Strategy

### UI Handling
- Uses `AOToast` for user-facing notifications
- Upload cancellation handled via AbortController detection
- Upload failures mark the file status as `Failed`

### Why this strategy
- Keeps UX responsive and explainable
- Avoids silent failures in long-running uploads
- Provides deterministic UI states for success/fail/cancel

---

## Design Principles

- **Provider-agnostic UI**: DraftEdit does not branch on provider; backend/service layer handles routing
- **Resilience-first attachment handling**: chunk uploads avoid timeouts and improve reliability
- **Optimistic UI with reconciliation**: progress shown immediately, server ids resolved afterward
- **Separation of concerns**: UI orchestrates; EmailService executes; MailContext maintains shared state

---

## Mermaid Diagrams

### Overall Flowchart (high-level request lifecycle)

```mermaid
flowchart TD
  A["Open DraftEdit (selectedMail set)"] --> B["Prefill Formik + Editor"]
  B --> C{"User action?"}

  C -->|"Edit fields"| D["Set isChange=true"]
  D --> E["Auto-save every 30s via saveAsDraft"]
  E --> C

  C -->|"Upload attachment"| F{"File size <= 4MB?"}
  F -->|"Yes"| G["postAttachment (multipart)"]
  F -->|"No"| H["uploadAttachmentInChunks"]
  H --> I["Resolve attachmentId via getMessage backoff"]
  G --> J["Update values.attachments"]
  I --> J
  J --> C

  C -->|"Send"| K["saveAsDraft (final)"]
  K --> L["sendMessage"]
  L --> M["Update folder count + removeEmail + clear selection"]
Sequence Diagram (UI → API → External Service)
mermaid
Copy code
sequenceDiagram
  participant UI as DraftEdit UI
  participant ES as EmailService
  participant API as Emails API
  participant P as Provider (Outlook/Gmail)

  UI->>ES: saveAsDraft(values)
  ES->>API: POST/PUT draft save
  API->>P: Provider-specific save draft
  P-->>API: OK
  API-->>ES: status
  ES-->>UI: update draftTime

  UI->>ES: uploadAttachment(...)
  ES->>API: POST attachments OR upload session/chunks
  API->>P: Provider upload (Graph/Gmail)
  P-->>API: attachment stored
  API-->>ES: response/progress
  ES-->>UI: progress + success

  UI->>ES: sendMessage(values)
  ES->>API: POST send
  API->>P: Provider send
  P-->>API: sent OK
  API-->>ES: status
  ES-->>UI: success + update MailContext
Update / Patch Flow
mermaid
Copy code
flowchart LR
  A["User edits field"] --> B["setIsChange(true)"]
  B --> C["Auto-save interval tick (30s)"]
  C --> D["EmailService.saveAsDraft"]
  D --> E["Draft saved time updated"]
  E --> F["setIsChange(false)"]
Attachment Flow
mermaid
Copy code
flowchart TD
  A["User drops file"] --> B{"Size <= 4MB?"}
  B -->|"Yes"| C["EmailService.postAttachment (FormData)"]
  B -->|"No"| D["EmailService.uploadAttachmentInChunks"]
  D --> E["Upload session + chunks + finalize"]
  E --> F["Resolve attachmentId via getMessage"]
  C --> G["Append returned attachments"]
  F --> G
  G --> H["UI shows attachments list + delete option"]
Final Outcome
This design achieves:

A consistent, validated draft-editing experience across providers

Reliable auto-save to protect user edits

Robust attachment upload support for both small and large files

Clean separation between UI orchestration, service calls, and shared mail state

Benefits:

UI: predictable UX, progress feedback, minimal provider branching

API: centralized provider routing and reusable endpoints

Scalability: easy to extend to Gmail as it reaches feature parity