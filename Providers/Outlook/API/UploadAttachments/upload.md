# Attachments Upload

## Overview

Attachments upload enables users to **add files to an email draft/message** (for example, while composing, replying, or forwarding). This feature exists to support a smooth compose experience across providers (Outlook now, Gmail later) while keeping the frontend **provider-agnostic**.

Why we implement attachments upload this way:

- **Provider-agnostic API**: the frontend always calls the same `Emails` controller endpoints. The backend routes to Outlook/Gmail using the authenticated user’s `EmailProvider` flag.
- **Performance + reliability**: large uploads are more likely to fail on slow networks or time out in one request, so we support a resumable/chunked workflow.
- **Microsoft Graph constraints**: Outlook attachments support both direct upload (small) and upload sessions (large). Our design maps cleanly to those capabilities.

Uploads are handled in **two modes**:

- **Single request upload** using `PostAttachments` (one call, simple)
- **Chunked upload** using `CreateUploadSession + UploadChunk + Status` (resumable, reliable)

> 📌 Note: Your workflow requirement here is followed exactly:  
> **> 4 MB → Single request (`PostAttachments`)**  
> **< 4 MB → Chunk upload session**  
> (This is the reverse of the usual approach, but documented as requested.)

---

## Limits

- **Single request (`PostAttachments`)**: up to **30 MB** (as per your stated max)
- **Chunked upload session**: up to **4 MB**
- **Chunk size**: 4 MiB (used by chunk workflow)

---

## Provider Routing

All attachment endpoints live inside the **single** `Emails` controller.

Routing logic:
1. Extract authenticated user from Bearer token
2. Load access token from DB
3. Check `accessToken.EmailProvider`
   - `Outlook` → route to `MicrosoftController`
   - `Gmail` → route to `GoogleController` (in progress)

Frontend does not need to switch endpoints by provider.

---

## Workflow Diagram

```mermaid
flowchart TD
  A["User selects attachment(s)"] --> B{"File size > 4 MB?"}

  B -- "Yes" --> C["Single request upload<br/>POST /Message/{messageId}/Attachments<br/><a href='./PostAttachments.md'>PostAttachments.md</a>"]
  C --> D["Emails Controller<br/>Resolve Provider"]
  D --> E["Outlook Provider<br/>MicrosoftController.PostAttachments"]
  D --> F["Gmail Provider<br/>GoogleController.PostGmailAttachments"]
  E --> G["Return attachment metadata"]
  F --> G

  B -- "No" --> H["Chunked upload workflow<br/><a href='./uploadAttachmentInChunks.md'>uploadAttachmentInChunks.md</a>"]
  H --> I["Create session<br/>POST /Message/{messageId}/Attachments/UploadSession"]
  I --> J["Receive sessionId + chunkSize"]
  J --> K["Upload chunk(s)<br/>PUT /Attachments/UploadSession/{sessionId}/Chunk"]
  K --> L["Finalize / confirm<br/>GET /Attachments/UploadSession/{sessionId}/Status"]
  L --> M["Return completion / progress"]


