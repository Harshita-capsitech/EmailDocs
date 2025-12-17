# Post Attachments (Single Request Upload)

## Overview

The Post Attachments API allows users to upload one or more files directly to an existing email message or draft. This is primarily used during Compose, Reply, or Forward flows, after a draft message has already been created.

This API is part of the unified Emails controller and is designed to be provider-agnostic. The frontend always calls the same endpoint, while the backend dynamically routes the request to the correct provider implementation (Outlook or Gmail) based on the authenticated user's configuration.

## Why We Use Post Attachments

The Post Attachments flow exists to support simple, direct file uploads without the overhead of chunk management or resumable sessions.

Key reasons for using this approach:

* Keeps the frontend independent of email providers
* Provides a fast and straightforward upload path
* Maps cleanly to Microsoft Graph FileAttachment APIs
* Ideal for standard attachment use cases
* Returns normalized attachment metadata for consistent UI handling

This method complements the chunked upload workflow and is intentionally kept minimal.

## When This API Is Used

This API is used when:

* A valid `messageId` already exists
* The user selects one or more files in the compose UI
* Attachments should be added immediately
* Resumable or chunked upload is not required

For resumable uploads or advanced reliability, the Upload Session (chunked) workflow is used instead.

## Endpoint
```
POST /Message/{messageId}/Attachments
```

## Authorization

* Authentication required
* Allowed roles:
   * ADMIN
   * MANAGER
   * STAFF

## Headers
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

## Request Parameters

### Path Parameter

* **messageId** - The ID of the email message or draft to which attachments will be added.

### Form Data

* **files** - One or more files uploaded as multipart form-data.

## High-Level Workflow

1. User selects attachment(s) in the email UI
2. Frontend sends a multipart request to the Emails controller
3. Backend validates messageId, files, and authentication
4. Backend resolves the email provider using the user's access token
5. Provider-specific attachment upload logic is executed
6. Uploaded attachment metadata is returned to the frontend

## Provider Routing

All attachment uploads are handled through the Emails controller.

Routing logic:

* Extract userId from Bearer token
* Load access token from database
* Check `EmailProvider` value
   * Outlook → route to `MicrosoftController`
   * Gmail → route to `GoogleController`

The frontend does not switch endpoints based on provider.

## Outlook Implementation

### Internal Method
```
MicrosoftController.PostAttachments
```

### Processing Steps

For each uploaded file:

1. Open the file stream
2. Read the file contents into memory
3. Create a Microsoft Graph `FileAttachment`
4. Upload the attachment to the target message
5. Normalize the provider response into a shared attachment model

### Microsoft Graph Call
```csharp
client.Me.Messages[messageId].Attachments.PostAsync(fileAttachment)
```

### Returned Attachment Data

Each successfully uploaded attachment includes:

* Attachment ID
* Content ID (CID)
* File name
* File size
* Content type
* Attachment content bytes (as returned by provider)

This normalized response ensures consistent behavior across providers.

## Success Response
```
200 OK
```

Response body includes:

* Confirmation message
* List of uploaded attachments with metadata

## Error Handling

* **400** – Message ID missing or no files uploaded
* **401** – Access token not found or invalid
* **400** – Unsupported email provider
* **500** – Provider failure or unexpected server error

## Design Notes

* Files are uploaded in a single request
* Entire file is read into memory
* Best suited for simple attachment scenarios
* Complements chunked upload workflows
* Provider-specific logic remains isolated in backend controllers

## Related Documentation

* Attachments Upload Overview
* Chunked Attachments Upload
* Emails Controller
* Outlook Provider Integration