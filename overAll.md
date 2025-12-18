# Acting Office Email System - Technical Documentation

## Table of Contents
1. [Overview](#1-overview)
2. [Data Flow Diagram (DFD)](#2-data-flow-diagram-dfd)
3. [Process Flow](#3-process-flow)
4. [ER Diagram](#4-er-diagram)
5. [Entity Definition](#5-entity-definition)
6. [Authentication](#6-authentication)
7. [APIs](#7-apis)
8. [Testing Guide](#8-testing-guide)
9. [References](#9-references)

---

## 1. Overview

### 1.1 System Introduction

The Acting Office Email System is a multi-provider email management platform that integrates with both Microsoft Outlook and Google Gmail. The system comprises two primary modules with distinct backend services but similar frontend capabilities.

### 1.2 Architecture Components

#### Backend Services

| Service | Purpose | Responsibilities |
|---------|---------|------------------|
| **Acting Office APIs** | Authentication & Communication Backend | • User authentication and authorization<br>• Email provider linking/unlinking operations<br>• Communication (Scheduling) module backend<br>• Token management and storage<br>• Scheduled email delivery tracking<br>• Retry logic for failed deliveries |
| **Acting Office Email Service** | Real-time Email Operations Backend | • Real-time email operations (inbox, sent, drafts)<br>• Email retrieval and display<br>• Email sending and management<br>• Provider-specific API routing (Outlook/Gmail)<br>• Email filtering and categorization |

#### Frontend Modules

| Module | Route | Backend Service | Features |
|--------|-------|----------------|----------|
| **Emails Module** | `/admin/emails` | Acting Office Email Service | • View inbox, sent items, drafts<br>• Send and receive emails<br>• Organize folders/labels<br>• Search and filter emails<br>• Immediate email operations |
| **Communication Module** | `/admin/communication` | Acting Office APIs | • **Same features as Emails Module**<br>• View inbox, sent items, drafts<br>• Send and receive emails<br>• **Additional:** Schedule emails for future delivery<br>• **Additional:** Track scheduled email status<br>• **Additional:** Automatic retry on failures |

### 1.3 Key Module Characteristics

**Emails Module:**
- Backend: Acting Office Email Service
- Purpose: Real-time email management
- Provider Support: Both Outlook and Gmail
- Features: Standard email operations (view, send, organize, search)

**Communication Module:**
- Backend: Acting Office APIs Service
- Purpose: Email management with scheduling capabilities
- Provider Support: Both Outlook and Gmail
- Features: **All features from Emails Module** + scheduling, tracking, and retry logic
- Unique Capability: Schedule emails for future delivery with automatic retry on failures

### 1.4 System Architecture Diagram

```mermaid
graph TB
    subgraph "Frontend Layer"
        A[User Login]
        B[Email Module]
        C[Communication Module]
    end

    subgraph "Backend Services"
        D[Acting Office APIs]
        E[Email Orchestration Service]
    end

    subgraph "External Email Providers"
        F[Microsoft Outlook Graph API]
        G[Google Gmail API]
    end

    A edge1@-->|Auth Success| B
    A edge2@-->|Auth Success| C

    B edge3@-->|Email Actions| E
    C edge4@-->|Email Operations + Scheduling| D

    E edge5@-->|Resolve Provider| D
    E edge6@-->|Outlook Mail Flow| F
    E edge7@-->|Gmail Mail Flow| G

    D edge8@-->|Account Link Unlink| F
    D edge9@-->|Account Link Unlink| G
    D edge10@-->|Email Operations| F
    D edge11@-->|Email Operations| G

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
    edge9@{ animate: fast }
    edge10@{ animate: fast }
    edge11@{ animate: fast }

```

---

## 2. Data Flow Diagram (DFD)

### 2.1 Level 0 - Context Diagram

```mermaid
graph LR
    subgraph External Entities
        U[User]
        O[Outlook API]
        G[Gmail API]
    end

    subgraph System
        S[Acting Office Email System]
    end

    U edge1@-->|Email Operations Requests| S
    S edge2@-->|Email Data Responses| U
    
    S edge3@-->|API Calls| O
    O edge4@-->|Email Data| S
    
    S edge5@-->|API Calls| G
    G edge6@-->|Email Data| S

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
```

### 2.2 Level 1 - Email Module Data Flow

```mermaid
graph LR
    FE[Frontend Email Section] edge1@--> ES[Acting Office Email Service]
    ES edge2@--> C{Check Provider}

    C edge3@-->|Outlook| F[Microsoft Graph API]
    C edge4@-->|Gmail| G[Gmail API]

    F edge5@--> H[Return Email Data]
    G edge6@--> H

    H edge7@--> ES
    ES edge8@--> FE

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }

```

### 2.3 Level 1 - Communication Module Data Flow

```mermaid
graph TD
    U[User] edge1@--> FE[Frontend Communication]
    FE edge2@--> API[Acting Office APIs]
    
    API edge3@--> DB[(Database)]
    DB edge4@--> API
    
    API edge5@--> Q[Email Queue]
    Q edge6@--> S[Scheduler]
    
    S edge7@--> P{Provider Check}
    P edge8@-->|Outlook| O[Outlook API]
    P edge9@-->|Gmail| G[Gmail API]
    
    O edge10@-->|Success| API
    G edge11@-->|Success| API
    
    O edge12@-->|Failure| NS[Notification Service]
    G edge13@-->|Failure| NS
    
    NS edge14@--> U
    API edge15@--> FE
    FE edge16@--> U

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
    edge9@{ animate: fast }
    edge10@{ animate: fast }
    edge11@{ animate: fast }
    edge12@{ animate: fast }
    edge13@{ animate: fast }
    edge14@{ animate: fast }
    edge15@{ animate: fast }
    edge16@{ animate: fast }

```

### 2.4 Provider Initialization Data Flow

```mermaid
flowchart TD
    A[Admin Initialize User] edge1@--> B[Initialize Create User]
    B edge2@--> C[Initialize Practice Id]
    C edge3@--> D[Set Email Provider]
    D edge4@--> E[Store User Configuration]
    E edge5@--> F[User Account Created]
    F edge6@--> G[Email Provider Configured Not Linked]

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }

```

---

## 3. Process Flow

### 3.1 User Login and Routing

```mermaid
flowchart TD
    A[User Login] edge1@--> B{Check Login Response}
    B edge2@--> C{Email Provider Is Google}
    
    C edge3@-->|Yes Google| D[Navigate Admin Emails Google]
    C edge4@-->|No Outlook| E[Navigate Admin Emails Outlook]
    
    E edge5@--> F{Check Link Status}
    F edge6@-->|Unlinked| G[Show Link Button]
    F edge7@-->|Linked| H[Show Unlink Button]
    
    D edge8@--> F

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }

```

**Login Routing Logic:**
- If provider is Google (0): Navigate to `/admin/emails/google`
- If provider is Outlook: Navigate to `/admin/emails`
- Display Link button if account is unlinked
- Display Unlink button if account is linked

### 3.2 Microsoft Outlook Link Process

```mermaid
graph LR
    A[User Click Link Outlook] edge1@--> B[Frontend Request Link]
    B edge2@--> C[API Start Microsoft Link]

    C edge3@--> D[Azure AD Auth Challenge]
    D edge4@--> E[User Microsoft Login]
    E edge5@--> F[Azure Callback to API]

    F edge6@--> G[Fetch User Tokens]
    G edge7@--> H[Save New Subscription]

    H edge8@--> I[Create Inbox Subscription]
    I edge9@--> J[Create Sent Subscription]

    J edge10@--> K[Update User Tokens]
    K edge11@--> L[Requeue Failed Emails]

    L edge12@--> M[Redirect User Success]

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
    edge9@{ animate: fast }
    edge10@{ animate: fast }
    edge11@{ animate: fast }
    edge12@{ animate: fast }

```

**Process Steps:**
1. User initiates link from profile settings
2. System redirects to Azure AD authentication
3. User authorizes application permissions
4. System receives authorization callback
5. Creates email subscriptions (Inbox and Sent folders)
6. Re-queues failed emails from past 15 days
7. Redirects to profile with success message

### 3.3 Microsoft Outlook Unlink Process

```mermaid
graph LR

    subgraph Client
        U[User]
    end

    subgraph Backend
        API[Acting Office APIs]
    end

    subgraph Storage
        DB[(Database)]
    end

    subgraph Microsoft
        G[Microsoft Graph]
    end

    U edge1@--> |POST Microsoft Unlink| API

    API edge2@--> |GetAsync userId Outlook| DB
    DB edge3@--> |User Access Tokens| API

    API edge4@--> |Get Authenticated Client| G

    API edge5@--> |Get All Subscriptions| DB
    DB edge6@--> |Subscription List| API
    API edge7@--> |Delete Subscription by Id| G

    API edge8@--> |Delete All Subscriptions| DB
    API edge9@--> |Revoke SignIn Sessions| G
    API edge10@--> |Delete User Access Tokens| DB
    API edge11@--> |Set Email Status Not Connected| DB

    API edge12@--> |Unlink Successful| U

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
    edge9@{ animate: fast }
    edge10@{ animate: fast }
    edge11@{ animate: fast }
    edge12@{ animate: fast }

```

**Process Steps:**
1. User initiates unlink from profile
2. System retrieves user access tokens
3. Deletes all Microsoft subscriptions
4. Revokes user sign-in sessions
5. Removes authentication tokens from database
6. Updates user email status to "Not Connected"
7. Sends reconnection notification email

### 3.4 Communication Module - Scheduled Email Workflow

```mermaid
graph TD
    U[User] edge1@--> FE[Frontend]
    FE edge2@--> API[Acting Office APIs]
    API edge3@--> Q[Email Queue]
    API edge4@--> FE

    S[Scheduler] edge5@--> Q
    Q edge6@--> S
    S edge7@--> P[Email Provider]

    P edge8@-->|Success| API
    API edge9@--> U

    P edge10@-->|Failure| NS[Notification Service]
    NS edge11@--> U
    S edge12@--> Q

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
    edge9@{ animate: fast }
    edge10@{ animate: fast }
    edge11@{ animate: fast }
    edge12@{ animate: fast }

```

**Workflow Steps:**
1. User creates scheduled email in Communication module
2. Email added to queue with scheduled time
3. Scheduler monitors queue for pending emails
4. At scheduled time, attempts to send email
5. On success: Deletes email from queue
6. On failure: Sends notification to user and retries automatically

### 3.5 Communication Flow States

```mermaid
stateDiagram-v2
    [*] --> Created: User creates scheduled email
    Created --> Queued: Email added to queue
    Queued --> Processing: Scheduler picks up email
    Processing --> Sent: Send successful
    Processing --> Failed: Send failed
    Failed --> Retry: Automatic retry
    Retry --> Processing: Retry attempt
    Retry --> PermanentFailure: Max retries exceeded
    Sent --> [*]: Email delivered & deleted from queue
    PermanentFailure --> [*]: User notified
```

### 3.6 Token Refresh & Error Handling

```mermaid
flowchart TD

    %% ───────────── Phase 1 : Request Entry ─────────────
    subgraph Phase1["Request Validation"]
        A[API Request] edge1@--> B{Token Valid}
    end

    %% ───────────── Phase 2 : Token Handling ─────────────
    subgraph Phase2["Token Handling"]
        B edge2@-->|Yes| C[Execute Request]
        B edge3@-->|No| D{Refresh Available}

        D edge4@-->|Yes| E[Refresh Token]
        E edge5@--> F{Refresh Success}

        F edge6@-->|Yes| C
        F edge7@-->|No| G[Need Approval]

        D edge8@-->|No| G
    end

    %% ───────────── Phase 3 : User Action ─────────────
    subgraph Phase3["User Re-Link Flow"]
        G edge9@--> H[Send Notification]
        H edge10@--> I[User Re Link]
    end

    %% ───────────── Phase 4 : Execution Result ─────────────
    subgraph Phase4["Request Result"]
        C edge11@--> J{Request Success}
        J edge12@-->|Yes| K[Return Data]
        J edge13@-->|No| L{Error Type}

        L edge14@-->|Auth Error| G
        L edge15@-->|Other Error| M[Return Error]
    end

    %% ───────────── Animations ─────────────
    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
    edge9@{ animate: fast }
    edge10@{ animate: fast }
    edge11@{ animate: fast }
    edge12@{ animate: fast }
    edge13@{ animate: fast }
    edge14@{ animate: fast }
    edge15@{ animate: fast }

```

**Token Management Strategy:**
- Automatic token validation before each request
- Proactive refresh before expiration (8-minute buffer for Google)
- User notification sent when re-authorization required
- Failed operations queued for retry after re-linking

### 3.7 Inbox Retrieval Process Flow

```mermaid
graph LR
    FE[Frontend Inbox] edge1@--> ES[Email Service or APIs]

    ES edge2@--> P{Email Provider}

    P edge3@-->|Outlook| MS[Microsoft Graph API]
    P edge4@-->|Gmail| GM[Gmail API]

    MS edge5@--> ES
    GM edge6@--> ES

    ES edge7@--> FE

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
```

**Note:** 
- **Email Module:** Routes through Acting Office Email Service
- **Communication Module:** Routes through Acting Office APIs (provides same features)

---

## 4. ER Diagram

### 4.1 Core Entity Relationships

```mermaid
erDiagram
    ApplicationUserAccessTokens ||--o{ ApplicationUser : "belongs to"
    ApplicationUserAccessTokens ||--o{ UserGoogleAccessToken : "contains"
    ApplicationUserAccessTokens ||--o{ UserMicrosoftAccessToken : "contains"
    ApplicationPractices ||--o{ ApplicationUserAccessTokens : "has many"
    ApplicationUserAccessTokens ||--o{ EmailQueueItem : "creates"
    
    ApplicationUserAccessTokens {
        string Id PK
        IdNameModel User
        enum EmailProvider
        UserGoogleAccessToken Google
        UserMicrosoftAccessToken Microsoft
        DateTime LastTokenRefresh
        string EmailAddress
        DateTime GoogleWatchUpdate
        ulong GmailHistoryId
    }
    
    UserMicrosoftAccessToken {
        string TenantId
        string ObjectId
        string Environment
        string TokenCache
        string SubscriptionIdInbox
        string SubscriptionIdSent
        DateTime SubscriptionUpdateInbox
        DateTime SubscriptionUpdateSent
        enum Status
        string LastError
    }
    
    ApplicationPractices {
        int Id PK
        string Name
        string Email
        string AppUrl
        bool EnableCacheFirst
        int CacheExpirationTime
    }
    
    EmailQueueItem {
        string Id PK
        string UserId FK
        string Provider
        DateTime ScheduledTime
        string Status
        int RetryCount
    }
```

---

## 5. Entity Definition

### 5.1 ApplicationUserAccessTokens

Stores authentication tokens and provider-specific credentials for users.

**Collection Name:** `UserAccessTokens`

| Field | Type | Description |
|-------|------|-------------|
| `Id` | string | Primary Key |
| `User` | IdNameModel | User reference (Id, Name) |
| `EmailProvider` | enum | Provider type (Outlook=1, Gmail=0) |
| `Google` | UserGoogleAccessToken | Google token object |
| `Microsoft` | UserMicrosoftAccessToken | Microsoft token object |
| `LastTokenRefresh` | DateTime | Last token refresh timestamp |
| `EmailAddress` | string | User's email address |
| `GoogleWatchUpdate` | DateTime | Google watch notification update |
| `GmailHistoryId` | ulong | Gmail history tracking ID |
| `Meta` | Dictionary | Additional metadata |
| `PracticeId` | int | Practice reference |

**Purpose:** Central storage for OAuth tokens and provider-specific authentication data.

### 5.2 UserMicrosoftAccessToken

Stores Microsoft-specific authentication and subscription data.

| Field | Type | Description |
|-------|------|-------------|
| `TenantId` | string | Azure AD tenant identifier |
| `ObjectId` | string | Unique account ID |
| `Environment` | string | Identity provider (e.g., login.microsoftonline.com) |
| `TokenCache` | string | Serialized token cache |
| `SubscriptionIdInbox` | string | Inbox notification subscription ID |
| `SubscriptionIdSent` | string | Sent items notification subscription ID |
| `SubscriptionUpdateInbox` | DateTime | Inbox subscription last update |
| `SubscriptionUpdateSent` | DateTime | Sent subscription last update |
| `Status` | UserAccessTokenStatus | Token status (Active, Deleted, NeedApproval) |
| `LastError` | string | Last error message |

**Purpose:** Manages Microsoft Graph API authentication and webhook subscriptions.

### 5.3 UserGoogleAccessToken

Stores Google-specific authentication data.

| Field | Type | Description |
|-------|------|-------------|
| `AccessToken` | string | OAuth access token |
| `RefreshToken` | string | OAuth refresh token |
| `ExpiresInSeconds` | int | Token expiration duration |
| `IssuedUtc` | DateTime | Token issue timestamp (UTC) |
| `Status` | UserAccessTokenStatus | Token status |
| `LastError` | string | Last error message |

**Purpose:** Manages Gmail API authentication with OAuth 2.0 tokens.

### 5.4 ApplicationPractices

Stores practice-level configuration.

**Collection Name:** `ApplicationPractices`

| Field | Type | Description |
|-------|------|-------------|
| `Id` | int | Primary Key |
| `PracticeId` | int | Practice identifier |
| `Name` | string | Practice name |
| `Email` | string | Practice email address |
| `EnableCacheFirst` | bool | Enable cache priority |
| `CacheExpirationTime` | int | Cache expiration (seconds) |

**Purpose:** Organization/tenant configuration and settings.

### 5.5 EmailQueueItem

Stores scheduled emails for the Communication module.

**Collection Name:** `EmailQueueItems`

| Field | Type | Description |
|-------|------|-------------|
| `Id` | string | Primary Key |
| `UserId` | string | User who scheduled email |
| `PracticeId` | int | Practice reference |
| `Provider` | ApplicationEmailServiceProviders | Email provider |
| `ScheduledTime` | DateTime | Scheduled delivery time |
| `Recipients` | List<string> | Email recipients |
| `Subject` | string | Email subject |
| `Body` | string | Email body content |
| `Status` | string | Queue status (Queued, Processing, Sent, Failed) |
| `RetryCount` | int | Number of retry attempts |
| `ErrorType` | EmailQueueItemErrorTypes | Error classification |
| `CreatedBy` | AuditInfo | Creation audit info |

**Purpose:** Queue management for scheduled email delivery in Communication module.

### 5.6 EmailQueueItemError

Tracks failed email delivery attempts.

**Collection Name:** `EmailQueueItemErrors`

| Field | Type | Description |
|-------|------|-------------|
| `Id` | string | Primary Key |
| `UserId` | string | User reference |
| `PracticeId` | int | Practice reference |
| `Provider` | ApplicationEmailServiceProviders | Email provider |
| `ErrorType` | EmailQueueItemErrorTypes | Error type (Reauthorize, etc.) |
| `Date` | DateTime | Error occurrence date |
| `ErrorMessage` | string | Error details |
| `CreatedBy` | AuditInfo | Creation audit info |

**Purpose:** Track failed emails for retry after account re-linking (15-day retention).

### 5.7 Enumerations

#### ApplicationEmailServiceProviders
- `Gmail = 0`
- `Outlook = 1`

#### UserAccessTokenStatus
- `Active` - Token valid and operational
- `Deleted` - Token revoked or removed
- `NeedApproval` - User action required

#### EmailQueueItemErrorTypes
- `Reauthorize` - Authentication required
- `InvalidRecipient` - Invalid email address
- `QuotaExceeded` - Provider quota limit reached
- `NetworkError` - Network connectivity issue
- `Unknown` - Unclassified error

#### ApplicationUserEmailStatus
- `NotConnected` - No provider linked
- `Connected` - Provider successfully linked
- `NeedApproval` - Re-authorization required

---

## 6. Authentication

### 6.1 Authentication Overview

The system uses OAuth 2.0 / OpenID Connect for email provider authentication:

**Microsoft Outlook:**
- Protocol: Azure AD OpenID Connect
- Authentication Flow: Authorization Code Flow
- Token Management: Access tokens with refresh capability
- Subscription Support: Webhook subscriptions for real-time notifications

**Google Gmail:**
- Protocol: OAuth 2.0
- Authentication Flow: Web Server Flow
- Token Management: Access tokens with refresh capability
- Notification Support: Push notifications via Pub/Sub

### 6.2 Authentication Flow

**Common Process for Both Providers:**

1. **Initiation:** User clicks "Link Account" button in profile settings
2. **Redirect:** System redirects to provider authentication page
3. **Authorization:** User authorizes Acting Office application
4. **Callback:** Provider redirects back with authorization code
5. **Token Exchange:** System exchanges code for access/refresh tokens
6. **Storage:** Tokens stored in ApplicationUserAccessTokens collection
7. **Status Update:** User email status updated to "Connected"
8. **Subscription:** Email subscriptions/watches created (if applicable)

### 6.3 Microsoft Authentication Process

**Link Account Steps:**
1. System initiates Azure AD authentication challenge
2. User redirected to Microsoft login page
3. User enters credentials and grants permissions
4. Azure AD redirects to callback endpoint with auth code
5. System exchanges code for tokens via Microsoft Identity platform
6. Creates Graph API subscriptions for Inbox and Sent folders
7. Re-queues failed emails from past 15 days
8. Redirects user to profile with success message

**Unlink Account Steps:**
1. System retrieves user access tokens from database
2. Deletes all Microsoft Graph subscriptions
3. Revokes user sign-in sessions via Graph API
4. Removes authentication tokens from database
5. Updates user email status to "Not Connected"
6. Sends reconnection notification email to user

### 6.4 Google Authentication Process

**Link Account Steps:**
1. System initiates OAuth 2.0 authorization flow
2. User redirected to Google login page
3. User grants permissions to Acting Office
4. Google redirects to callback with authorization code
5. System exchanges code for access and refresh tokens
6. Creates Gmail watch for push notifications
7. Stores tokens in database
8. Updates user status to "Connected"

**Unlink Account Steps:**
1. System retrieves Google access tokens
2. Stops Gmail watch notifications
3. Revokes OAuth tokens via Google API
4. Removes tokens from database
5. Updates user status to "Not Connected"
6. Sends reconnection notification

### 6.5 Token Management

**Token Lifecycle:**
- **Issuance:** Tokens issued during initial authentication
- **Storage:** Encrypted storage in MongoDB with practice-level isolation
- **Caching:** 2-hour distributed cache for performance
- **Refresh:** Automatic refresh before expiration (8-minute buffer for Google)
- **Revocation:** Manual unlink or automatic on authentication errors
- **Re-authorization:** User notified when re-link required

**Token Status Types:**
- **Active:** Token valid and operational
- **Deleted:** Token revoked (manual unlink or error)
- **NeedApproval:** User must re-authorize

**Error Handling:**
- Authentication failures trigger status change to "NeedApproval"
- User receives email notification with re-link instructions
- Failed emails (with ErrorType.Reauthorize) retained for 15 days
- Upon re-linking, failed emails automatically re-queued

### 6.6 Security Measures

**Token Security:**
- Encrypted storage in MongoDB
- Practice-level data isolation
- Secure token refresh mechanism
- Automatic expiration and rotation

**Access Control:**
- Role-based authorization (ADMIN, MANAGER, STAFF)
- JWT-based API authentication
- User-specific token access
- Cross-user access prevention

**Account Protection:**
- Sign-in session revocation on unlink
- Subscription cleanup on disconnect
- Audit trail for link/unlink operations
- Email notifications for account changes

---

## 7. APIs

### 7.1 API Overview

The Acting Office Email System provides RESTful APIs organized into two main categories:

**Authentication APIs (Acting Office APIs Service):**
- Microsoft/Google account linking
- Account unlinking
- Token management
- Status updates

**Email Operation APIs:**
- Inbox retrieval (Email Service for Email Module)
- Email operations with scheduling (APIs Service for Communication Module)
- Email sending and management
- Search and filtering

### 7.2 Microsoft Outlook API Endpoints

**Authentication Endpoints:**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/Microsoft/Link` | GET | Initiates OAuth flow to link Outlook account |
| `/Microsoft/AuthCallback` | GET | Handles OAuth callback after authentication |
| `/Microsoft/Unlink` | POST | Unlinks Outlook account and removes subscriptions |

**Key Features:**
- Anonymous access for Link and Callback (OAuth flow)
- Role-based authorization for Unlink (ADMIN, MANAGER, STAFF)
- Automatic subscription management
- Failed email re-queuing on successful link

### 7.3 Google Gmail API Endpoints

**Authentication Endpoints:**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/Google/Link` | GET | Initiates OAuth 2.0 flow to link Gmail account |
| `/Google/AuthCallback` | GET | Handles OAuth callback |
| `/Google/Unlink` | POST | Unlinks Gmail account and stops watch |

**Key Features:**
- OAuth 2.0 authorization flow
- Push notification watch management
- Token refresh handling
- Account status synchronization

### 7.4 Email Service API Endpoints

**Email Module (Acting Office Email Service):**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/Emails/Inbox` | GET | Retrieve inbox emails with filtering |
| `/Emails/Send` | POST | Send email immediately |
| `/Emails/Draft` | POST | Save email as draft |
| `/Emails/Delete` | DELETE | Delete email |
| `/Emails/Move` | POST | Move email to folder |
| `/Emails/Search` | GET | Search emails |

**Communication Module (Acting Office APIs):**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/Communication/Inbox` | GET | Retrieve inbox with same features as Email Module |
| `/Communication/Send` | POST | Send email immediately |
| `/Communication/Schedule` | POST | Schedule email for future delivery |
| `/Communication/Scheduled` | GET | Get list of scheduled emails |
| `/Communication/Cancel` | DELETE | Cancel scheduled email |

### 7.5 Common API Features

**Authentication:**
- JWT Bearer token authentication
- Role-based access control
- User context validation

**Filtering and Pagination:**
- Start/length pagination
- View type filters (unread, read, all)
- Importance/starred filters
- Attachment filters
- Date range filters
- Category filters
- Sort options

**Provider Routing:**
- Automatic provider detection from user tokens
- Dynamic routing to Outlook or Gmail APIs
- Unified response format regardless of provider

**Error Handling:**
- Standardized error response structure
- HTTP status codes
- Detailed error messages
- Error type classification

### 7.6 Provider Comparison

| Feature | Microsoft Outlook | Google Gmail |
|---------|------------------|--------------|
| **Provider Value** | 1 | 0 |
| **Auth Method** | Azure AD OpenID Connect | OAuth 2.0 |
| **API** | Microsoft Graph API | Gmail API |
| **Real-time Notifications** | Webhook subscriptions | Push notifications |
| **Token Expiration** | Standard OAuth | 8-minute buffer |
| **Supported Features** | All email operations + scheduling | All email operations + scheduling |

### 7.7 Response Format

All APIs return responses in a standardized format:

**Success Response:**
- `result`: Response data (type varies by endpoint)
- `isSuccess`: true
- `errors`: Empty array

**Error Response:**
- `result`: null or partial data
- `isSuccess`: false
- `errors`: Array of error objects with code and message

---

## 8. Testing Guide

### 8.1 Testing Strategy

**Test Levels:**
1. Unit Testing - Individual components and functions
2. Integration Testing - API endpoints and database operations
3. End-to-End Testing - Complete user workflows
4. Performance Testing - Load and stress testing
5. Security Testing - Authentication and authorization

### 8.2 Key Test Scenarios

#### Authentication Testing

**Scenario 1: Link Microsoft Outlook Account**
- Objective: Verify successful account linking
- Steps: Login → Navigate to profile → Click Link button → Authorize → Verify success
- Expected: Token stored, status "Connected", subscriptions created

**Scenario 2: Unlink Microsoft Outlook Account**
- Objective: Verify successful account unlinking
- Steps: Click Unlink → Confirm → Verify status update
- Expected: Subscriptions deleted, tokens removed, notification sent

**Scenario 3: Link Google Gmail Account**
- Objective: Verify Gmail account linking
- Steps: Similar to Outlook with Google OAuth flow
- Expected: Token stored, watch created, status updated

#### Email Module Testing

**Scenario 4: View Inbox**
- Objective: Verify inbox retrieval for both providers
- Steps: Navigate to emails → View inbox → Apply filters
- Expected: Emails displayed, filters work, pagination functional

**Scenario 5: Send Email**
- Objective: Verify immediate email sending
- Steps: Compose email → Add recipients → Send
- Expected: Email sent successfully, appears in sent folder

**Scenario 6: Search and Filter**
- Objective: Verify search and filter functionality
- Steps: Apply various filters and search terms
- Expected: Relevant results returned, filters applied correctly

#### Communication Module Testing

**Scenario 7: Schedule Email**
- Objective: Verify email scheduling functionality
- Steps: Navigate to communication → Schedule email → Set future time
- Expected: Email queued, appears in scheduled list

**Scenario 8: Scheduled Email Delivery**
- Objective: Verify scheduled email sent at correct time
- Steps: Wait for scheduled time → Verify delivery
- Expected: Email sent at scheduled time, removed from queue

**Scenario 9: Failed Email Retry**
- Objective: Verify automatic retry logic
- Steps: Unlink account → Schedule email → Re-link → Verify retry
- Expected: Failed email re-queued and sent after re-link

#### Token Management Testing

**Scenario 10: Token Refresh**
- Objective: Verify automatic token refresh
- Steps: Use expired/near-expired token → Make API request
- Expected: Token automatically refreshed, request succeeds

**Scenario 11: Token Expiration Handling**
- Objective: Verify handling of expired tokens
- Steps: Use expired token → Verify notification sent
- Expected: User notified to re-link account

### 8.3 Performance Testing

**Load Test Scenarios:**

1. **Concurrent Inbox Requests**
   - 100 concurrent users requesting inbox
   - Target response time: < 2 seconds
   - Success rate: > 99%

2. **Bulk Email Scheduling**
   - Schedule 1000 emails across multiple users
   - Verify queue processing efficiency
   - Monitor system resource usage

3. **Token Refresh Under Load**
   - Multiple users with near-expired tokens
   - Simultaneous requests triggering refresh
   - Verify all refreshes succeed

### 8.4 Security Testing

**Test Cases:**

1. **Unauthorized Access Prevention**
   - Attempt access without authentication
   - Expected: 401 Unauthorized

2. **Cross-User Access Prevention**
   - User A tries to access User B's emails
   - Expected: 403 Forbidden

3. **Token Validation**
   - Use invalid or expired tokens
   - Expected: Proper error handling

4. **Input Validation**
   - Test with malicious inputs
   - Expected: Input sanitized, no vulnerabilities

### 8.5 Testing Tools

**Recommended Tools:**
- Postman/Insomnia - API testing
- JMeter/k6 - Load testing
- Selenium - End-to-end testing
- xUnit/NUnit - Unit testing (.NET)
- MongoDB Compass - Database verification

### 8.6 Test Data Management

**Test Accounts:**
- Create dedicated test accounts for Outlook and Gmail
- Use sandbox/test environments when available
- Maintain separate test practice IDs
- Clean up test data after testing

**Database Verification:**
- Monitor token storage and updates
- Verify queue item creation and deletion
- Check subscription/watch status
- Validate audit trails

---

## 9. References

### 9.1 External Documentation

**Microsoft Resources:**
- Microsoft Graph API Documentation
- Azure AD Authentication Documentation
- Outlook Mail API Reference
- Microsoft Identity Platform

**Google Resources:**
- Gmail API Documentation
- Google OAuth 2.0 Documentation
- Gmail Push Notifications Guide
- Google Cloud Platform Console

### 9.2 Technology Stack

| Technology | Purpose |
|------------|---------|
| .NET 8.0 | Backend framework |
| MongoDB | Database storage |
| Redis | Distributed caching |
| Microsoft.Identity.Web | Azure AD authentication |
| Google.Apis.Gmail.v1 | Gmail API client |
| Microsoft.Graph | Graph API client |

### 9.3 Database Collections

| Collection | Purpose |
|------------|---------|
| `ApplicationPractices` | Practice/tenant configuration |
| `UserAccessTokens` | Authentication tokens |
| `ApplicationUserOutlookSubscription` | Outlook webhook subscriptions |
| `EmailQueueItems` | Scheduled emails queue |
| `EmailQueueItemErrors` | Failed email tracking |

### 9.4 Common Workflows Summary

| Workflow | Services Involved | User Impact |
|----------|------------------|-------------|
| User Login & Routing | Acting Office APIs | Automatic redirect based on provider |
| Link Email Account | Acting Office APIs → Provider OAuth | Enable email features |
| Unlink Email Account | Acting Office APIs → Provider API | Disable email features |
| View Inbox (Email Module) | Email Service → Provider API | Real-time email access |
| View Inbox (Communication Module) | Acting Office APIs → Provider API | Email access with scheduling |
| Send Email Immediately | Email Service or APIs → Provider API | Instant delivery |
| Schedule Email | Acting Office APIs → Queue → Provider API | Future delivery with retry |
| Token Refresh | Automatic (Background Service) | Seamless operation |
| Failed Email Recovery | Acting Office APIs (on re-link) | Automatic retry |

### 9.5 System Characteristics

**Scalability:**
- Practice-level data isolation
- Distributed caching for performance
- Queue-based scheduling architecture
- Horizontal scaling capability

**Reliability:**
- Automatic token refresh
- Failed email retry mechanism
- 15-day error retention
- Comprehensive error handling

**Security:**
- OAuth 2.0 / OpenID Connect
- Encrypted token storage
- Role-based access control
- Audit trail for operations

**Maintainability:**
- Modular architecture
- Clear separation of concerns
- Standardized API responses
- Comprehensive logging

---

*Document Version: 2.0*  
*Last Updated: December 2025*  
*Acting Office Email System - Technical Documentation*
