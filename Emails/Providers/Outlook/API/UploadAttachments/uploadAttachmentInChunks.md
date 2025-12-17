# Chunked Attachments Upload (Upload Session)

## Overview

The Chunked Attachments Upload workflow enables reliable uploading of email attachments using multiple smaller chunks instead of a single large request. This approach is used to safely upload files that may fail due to size limits, slow networks, or provider constraints.

This feature is implemented as part of the common Emails controller and is provider-agnostic. The frontend always calls the same APIs, while the backend dynamically routes requests to the appropriate provider implementation (Outlook currently, Gmail planned).

Chunked uploads are especially important for:

* Large files
* Unstable network connections
* Avoiding request timeouts
* Supporting resumable uploads
* Aligning with Microsoft Graph upload session requirements

## Why Chunked Upload Exists

Chunked upload is designed to solve problems that single-request uploads cannot reliably handle.

Key reasons for using this approach:

* **Resumability**: failed uploads can continue from where they stopped
* **Network tolerance**: smaller chunks reduce the risk of failure
* **Concurrency safety**: per-session locking prevents race conditions
* **Provider compatibility**: Microsoft Graph requires upload sessions for large attachments
* **Scalability**: supports future providers without frontend changes

## When This Workflow Is Used

This workflow is used when:

* A draft or message already exists (messageId)
* The file is uploaded in parts instead of a single request
* Reliability is prioritized over simplicity
* The frontend decides to use upload sessions

The frontend is responsible for deciding whether to use single upload or chunked upload.

## High-Level Workflow

1. Frontend requests an upload session
2. Backend creates a provider upload session
3. Session metadata is cached server-side
4. Frontend uploads file chunks sequentially
5. Backend forwards chunks to the provider
6. Frontend polls status until upload is complete
7. Backend finalizes and cleans up session state

## Step 1: Create Upload Session

### Endpoint
```
POST /Message/{messageId}/Attachments/UploadSession
```

### Authorization
```
Authorization: Bearer <token>
```

### Request Body

CreateChunkUploadRequest (JSON):

* `fileName`
* `size`
* `contentType`

### Workflow

1. Backend validates messageId and authentication
2. User access token is loaded
3. Provider is resolved using EmailProvider
4. For Outlook:
   * Microsoft Graph CreateUploadSession is called
   * Upload URL and metadata are returned
5. Internal session state is created and cached
6. A unique sessionId is generated and returned

### Response

* `sessionId`
* `chunkSize` (4 MiB)
* total file size
* file name

## Step 2: Upload Attachment Chunk

### Endpoint
```
PUT /Attachments/UploadSession/{sessionId}/Chunk
```

### Query Parameters

* `start` – starting byte index
* `end` – ending byte index
* `total` – total file size

### Headers
```
Authorization: Bearer <token>
Content-Type: application/octet-stream
```

### Workflow

1. Frontend splits the file into 4 MB chunks
2. Each chunk is uploaded sequentially
3. Backend validates the sessionId
4. Session state is loaded from cache
5. A per-session lock is acquired
6. Chunk is forwarded to the provider upload URL
7. Progress is recorded and returned
8. Lock is released

Only one chunk per session is processed at a time.

## Step 3: Upload Status / Finalization

### Endpoint
```
GET /Attachments/UploadSession/{sessionId}/Status
```

### Purpose

* Allows the frontend to verify upload completion
* Prevents guessing or premature finalization
* Ensures all chunks are committed successfully

### Workflow

1. Backend checks cached session state
2. If upload is complete, progress is returned
3. If session is missing or expired, a not-found response is returned

## Session State Management

Each upload session stores the following metadata in cache:

* MessageId
* UploadUrl (provider-specific)
* Total file size
* Expiration time
* File name
* Content type
* Bytes committed
* First chunk checksum (optional)

### Cache Keys

* `mail:attach:session:{sessionId}`
* `mail:attach:done:{sessionId}`

### Cache Lifetime

* **Absolute expiration**: 2 hours
* **Sliding expiration**: 30 minutes

These values allow large uploads on slow networks while preventing stale sessions.

## Concurrency Control

To prevent race conditions and duplicate uploads:

* A SemaphoreSlim lock is created per sessionId
* Only one chunk can be processed at a time
* Late or duplicate chunks are rejected safely

This ensures consistent progress tracking and prevents corruption.

## Cleanup Strategy

After upload completion:

* Session data is kept briefly to reject late chunks
* Cache entries are removed automatically
* Session locks are disposed safely

This prevents memory leaks and stale state.

## Provider Routing

All endpoints are implemented in the Emails controller.

Routing logic:

1. Load authenticated user
2. Read EmailProvider
   * Outlook → MicrosoftController
   * Gmail → GoogleController (stubbed)

The frontend does not change behavior based on provider.

## Error Handling

* **400** – Missing messageId or sessionId
* **401** – Access token missing or invalid
* **404** – Upload session not found or expired
* **5xx** – Provider failure or network error

Frontend should retry failed chunks and rely on the Status endpoint for final confirmation.

## Design Notes

* Uses Microsoft Graph upload sessions internally
* Provider logic isolated from controller
* Safe for concurrent users and large files
* Fully compatible with resumable uploads
* Designed for future Gmail implementation

## Related Documentation

* Attachments Upload Overview
* Post Attachments (Single Request Upload)
* Emails Controller
* Outlook Provider Integration