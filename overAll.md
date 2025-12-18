# Acting Office Email System - Technical Documentation

## Table of Contents
1. [Overview](#system-overview)
2. [Architecture Components](#architecture-components)
3. [Email Provider Integration](#email-provider-integration)
4. [Email Module Workflow](#email-module-workflow)
5. [Communication Module Workflow](#communication-module-workflow)
6. [Authentication & Token Management](#authentication--token-management)
7. [Data Models](#data-models)
8. [API Endpoints](#api-endpoints)

---

## 1. System Overview

The Acting Office Email System is a multi-provider email management platform that integrates with both Microsoft Outlook and Google Gmail. The system comprises two primary modules:

- **Email Module**: Handles real-time email operations (inbox, sent items, drafts)
- **Communication Module**: Manages scheduled email communications

### System Architecture

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
    C edge4@-->|Scheduled Communications| D

    E edge5@-->|Resolve Provider| D
    E edge6@-->|Outlook Mail Flow| F
    E edge7@-->|Gmail Mail Flow| G

    D edge8@-->|Account Link Unlink| F
    D edge9@-->|Account Link Unlink| G

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }
    edge9@{ animate: fast }

```

---

## 2. Architecture Components

### 2.1 Backend Services

| Service | Responsibility |
|---------|----------------|
| **Acting Office APIs** | - User authentication<br>- Email provider link/unlink operations<br>- Scheduled communication management<br>- Account status management |
| **Acting Office Email Service** | - Real-time email operations (inbox, sent, drafts)<br>- Email retrieval and sending<br>- Provider-specific API routing |

### 2.2 Database Collections

```mermaid
erDiagram
    ApplicationUserAccessTokens ||--o{ ApplicationUser : "belongs to"
    ApplicationUserAccessTokens ||--o{ UserGoogleAccessToken : "contains"
    ApplicationUserAccessTokens ||--o{ UserMicrosoftAccessToken : "contains"
    ApplicationPractices ||--o{ ApplicationUserAccessTokens : "has many"
    
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
```

---

## 3. Email Provider Integration

### 3.1 Provider Initialization Flow

When an admin creates a user, the system initializes the email provider configuration:

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

### 3.2 Login and Routing Logic

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

### 3.3 Provider Comparison

| Feature | Microsoft Outlook | Google Gmail |
|---------|------------------|--------------|
| **Provider Enum** | `ApplicationEmailServiceProviders.Outlook` (1) | `ApplicationEmailServiceProviders.Gmail` (0) |
| **Auth Method** | Azure AD OpenID Connect | OAuth 2.0 |
| **Token Storage** | `UserMicrosoftAccessToken` | `UserGoogleAccessToken` |
| **API** | Microsoft Graph API | Gmail API |
| **Subscription Support** | Yes (Inbox & Sent folders) | Watch notifications |

---

## 4. Email Module Workflow

### 4.1 Email Operations Architecture

```mermaid
graph LR
    A[Frontend Email Section] edge1@--> B[Acting Office Email Service]
    B edge2@--> C{Check Provider}

    C edge3@-->|Outlook| F[Microsoft Graph API]
    C edge4@-->|Gmail| G[Gmail API]

    F edge5@--> H[Return Email Data]
    G edge6@--> H

    H edge7@--> A

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }

```

### 4.2 Inbox Retrieval Flow

The system uses a unified controller with provider-specific routing:

```mermaid
graph LR
    FE[Frontend Inbox] edge1@--> ES[Email Service]

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

## 5. Communication Module Workflow

### 5.1 Scheduled Email System

The Communication module handles scheduled email sending through the Acting Office APIs service.

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

### 5.2 Communication Flow States

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

---

## 6. Authentication & Token Management

### 6.1 Microsoft Outlook Link Flow

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

### 6.2 Microsoft Outlook Unlink Flow

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



### 6.3 Token Refresh & Error Handling

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

## 8. API Endpoints

### 8.1 Microsoft Outlook Endpoints

| Endpoint | Method | Authentication | Description |
|----------|--------|----------------|-------------|
| `/Microsoft/Link` | GET | AllowAnonymous | Initiates OAuth flow to link Outlook account |
| `/Microsoft/AuthCallback` | GET | AllowAnonymous | OAuth callback after successful authentication |
| `/Microsoft/Unlink` | POST | Authorized (ADMIN,MANAGER,STAFF) | Unlinks Outlook account and removes subscriptions |

### 8.2 Email Service Endpoints

| Endpoint | Method | Parameters | Description |
|----------|--------|-----------|-------------|
| `/Inbox` | GET | See Email Controller Parameters | Retrieves email list based on provider |
| `/Send` | POST | Email content & recipients | Sends email through provider |
| `/Draft` | POST | Draft content | Saves email as draft |
| `/Delete` | DELETE | Email ID | Deletes email |
| `/Move` | POST | Email ID, folder | Moves email to folder |


## Appendix B: Common Workflows Summary

| Workflow | Services Involved | User Impact |
|----------|------------------|-------------|
| User Login & Routing | Acting Office APIs | Automatic redirect to appropriate email interface |
| Link Email Account | Acting Office APIs → Azure AD/Google OAuth | User can send/receive emails |
| Unlink Email Account | Acting Office APIs → Provider API | Email features disabled |
| View Inbox | Email Service → Provider API | See emails in unified interface |
| Send Email (Now) | Email Service → Provider API | Immediate delivery |
| Schedule Email | Communication Module → Acting Office APIs | Future delivery with retry |
| Token Refresh | Automatic (Email Service) | Seamless operation |
| Failed Email Recovery | Acting Office APIs (on re-link) | Automatic retry of failed emails |

---

*Document Version: 1.0*  
*Last Updated: December 2025*  
*Acting Office Email System*
