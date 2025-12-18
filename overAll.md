# Acting Office Email System - Technical Documentation

## Table of Contents
1. [System Overview](#system-overview)
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
sequenceDiagram
    participant Admin
    participant System
    participant DB
    participant User
    
    Admin->>System: Create User
    System->>DB: Create PracticeId
    System->>DB: Set Email Provider (0=Google, 1=Outlook)
    System->>DB: Store User Configuration
    System->>User: User Account Created
    Note over User: Email provider configured<br/>but not yet linked
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
    A[Frontend - Email Section] --> B[Acting Office Email Service]
    B --> C{Check Provider}
    C -->|Outlook| D[Microsoft Controller]
    C -->|Gmail| E[Google Controller]
    D --> F[Microsoft Graph API]
    E --> G[Gmail API]
    F --> H[Return Email Data]
    G --> H
    H --> A
```

### 4.2 Inbox Retrieval Flow

The system uses a unified controller with provider-specific routing:

```mermaid
sequenceDiagram
    participant FE as Frontend
    participant ES as Email Service
    participant DB as Database
    participant MC as Microsoft Controller
    participant GC as Google Controller
    participant API as External API
    
    FE->>ES: GET /Inbox (parameters)
    ES->>DB: GetByUserId(currentUserId)
    DB-->>ES: ApplicationUserAccessTokens
    
    alt EmailProvider is Outlook
        ES->>MC: GetMessages()
        MC->>API: Microsoft Graph API Request
        API-->>MC: Email Data
        MC-->>ES: Formatted Response
    else EmailProvider is Gmail
        ES->>GC: GetMessageList()
        GC->>API: Gmail API Request
        API-->>GC: Email Data
        GC-->>ES: Formatted Response
    end
    
    ES-->>FE: ApiResponse<PagedData<EmailsListItem>>
```

### 4.3 Email Controller Parameters

The unified email controller (`/Inbox` endpoint) supports the following parameters:

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `start` | int | Pagination start index | 0 |
| `length` | int | Number of items per page | 35 |
| `search` | string | Search query | null |
| `userId` | string | User identifier | "" |
| `inboxType` | string | Folder type (inbox, sent, drafts) | "inbox" |
| `isImportant` | bool | Filter important emails only | false |
| `viewType` | string | View filter (unread, read, all) | "" |
| `isFlagged` | bool? | Filter flagged emails | null |
| `isStarred` | bool? | Filter starred emails (Gmail) | null |
| `hasAttachment` | bool? | Filter emails with attachments | null |
| `hasMentioned` | bool? | Filter emails where user is mentioned | null |
| `categoryType` | string | Email category filter | null |
| `sortBy` | string | Sort field | "Date" |
| `nextPageToken` | string? | Gmail pagination token | null |
| `startDate` | string? | Date range start | null |
| `endDate` | string? | Date range end | null |

---

## 5. Communication Module Workflow

### 5.1 Scheduled Email System

The Communication module handles scheduled email sending through the Acting Office APIs service.

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API as Acting Office APIs
    participant Scheduler
    participant EmailQueue
    participant Provider
    participant NotificationService
    
    User->>Frontend: Create Scheduled Email
    Frontend->>API: POST /schedule
    API->>EmailQueue: Add to Queue
    API-->>Frontend: Schedule Confirmed
    
    loop Scheduler Monitoring
        Scheduler->>EmailQueue: Check Due Emails
        EmailQueue-->>Scheduler: Due Email Items
        Scheduler->>Provider: Send Email
        
        alt Send Success
            Provider-->>Scheduler: Success
            Scheduler->>API: DELETE /schedule/{id}
            Scheduler->>User: Success Notification
        else Send Failure
            Provider-->>Scheduler: Failure
            Scheduler->>NotificationService: Send Failure Email to User
            Scheduler->>EmailQueue: Retry Queue
            Note over Scheduler: System retries sending
        end
    end
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
sequenceDiagram
    participant User
    participant Frontend
    participant API as Acting Office APIs
    participant Azure as Azure AD
    participant Graph as Microsoft Graph
    participant DB
    
    User->>Frontend: Click 'Link Outlook'
    Frontend->>API: GET /Microsoft/Link?uid={userId}&practiceId={id}
    API->>Azure: Challenge with AuthenticationProperties
    Azure->>User: Redirect to Microsoft Login
    User->>Azure: Authenticate
    Azure->>API: GET /Microsoft/AuthCallback?uid={userId}&pid={practiceId}
    
    API->>DB: GetByUserId(userId)
    DB-->>API: ApplicationUserAccessTokens
    
    API->>API: SaveNewSubscription()
    API->>Graph: Create Inbox Subscription
    API->>Graph: Create Sent Folder Subscription
    
    API->>DB: Update UserAccessTokens
    API->>DB: Re-queue Previous Failed Emails (last 15 days)
    
    API-->>User: Redirect to Profile (success message)
```

### 6.2 Microsoft Outlook Unlink Flow

```mermaid
sequenceDiagram
    participant User
    participant API as Acting Office APIs
    participant Graph as Microsoft Graph
    participant DB
    
    User->>API: POST /Microsoft/Unlink
    API->>DB: GetAsync(userId, Outlook provider)
    DB-->>API: ApplicationUserAccessTokens
    
    API->>Graph: Get Authenticated Client
    
    loop For Each Subscription
        API->>DB: Get All Subscriptions
        DB-->>API: Subscription List
        API->>Graph: DELETE /Subscriptions/{id}
    end
    
    API->>DB: Delete All Subscriptions from DB
    API->>Graph: RevokeSignInSessions()
    API->>DB: Delete UserAccessTokens
    API->>DB: SetEmailStatus(NotConnected)
    
    API-->>User: ApiResponse (unlink successful)
```

### 6.3 Token Status Management

The system maintains token status using the `UserAccessTokenStatus` enum:

```mermaid
flowchart LR
    Start((Start)) edge1@--> Active[Active Account]

    Active edge2@--> Expired[Token Expired]
    Active edge3@--> Deleted[Account Deleted]
    Active edge4@--> Error[API Error]

    Expired edge5@--> Active
    Error edge6@--> Active

    Deleted edge7@--> Active
    Deleted edge8@--> End((End))

    edge1@{ animate: fast }
    edge2@{ animate: fast }
    edge3@{ animate: fast }
    edge4@{ animate: fast }
    edge5@{ animate: fast }
    edge6@{ animate: fast }
    edge7@{ animate: fast }
    edge8@{ animate: fast }

```

### 6.4 Token Refresh & Error Handling

```mermaid
flowchart TD
    A[API Request] edge1@--> B{Token Valid}

    B edge2@-->|Yes| C[Execute Request]
    B edge3@-->|No| D{Refresh Available}

    D edge4@-->|Yes| E[Refresh Token]
    E edge5@--> F{Refresh Success}

    F edge6@-->|Yes| C
    F edge7@-->|No| G[Need Approval]

    D edge8@-->|No| G

    G edge9@--> H[Send Notification]
    H edge10@--> I[User Re Link]

    C edge11@--> J{Request Success}
    J edge12@-->|Yes| K[Return Data]
    J edge13@-->|No| L{Error Type}

    L edge14@-->|Auth Error| G
    L edge15@-->|Other Error| M[Return Error]

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

---

## 7. Data Models

### 7.1 ApplicationUserAccessTokens

Primary model for storing user authentication tokens and email provider information.

```typescript
class ApplicationUserAccessTokens {
    Id: string;                              // Primary key
    User: IdNameModel;                       // User reference
    EmailProvider: ApplicationEmailServiceProviders; // 0=Gmail, 1=Outlook
    Google?: UserGoogleAccessToken;          // Gmail tokens
    Microsoft?: UserMicrosoftAccessToken;    // Outlook tokens
    LastTokenRefresh?: DateTime;             // Last refresh timestamp
    EmailAddress?: string;                   // User's email
    GoogleWatchUpdate?: DateTime;            // Gmail watch update date
    GmailHistoryId?: ulong;                  // Gmail sync point
    Meta?: Dictionary<string, string>;       // Additional metadata
    
    // Methods
    IsGoogleTokenExpired(): boolean;         // Check token expiration
}
```

### 7.2 UserMicrosoftAccessToken

```typescript
class UserMicrosoftAccessToken {
    TenantId?: string;                       // Azure AD tenant
    ObjectId?: string;                       // User object ID
    Environment?: string;                    // Identity provider URL
    TokenCache?: string;                     // Cached ID token
    SubscriptionIdInbox?: string;            // Inbox notification subscription
    SubscriptionIdSent?: string;             // Sent folder notification subscription
    SubscriptionUpdateInbox?: DateTime;      // Inbox subscription update time
    SubscriptionUpdateSent?: DateTime;       // Sent subscription update time
    Status: UserAccessTokenStatus;           // Token status
    LastError?: string;                      // Last error message
    
    // Methods
    GetHomeAccountId(): string;              // Returns "ObjectId.TenantId"
}
```

### 7.3 Database Operations

```mermaid
classDiagram
    class ApplicationUserAccessTokensDB {
        +string CollectionName
        +GetByUserId(userId: string)
        +UpdateGoogleToken(userId: string, token: UserGoogleAccessToken)
        +UpdateMicrosoftToken(userId: string, token: UserMicrosoftAccessToken)
        +DeleteGoogleToken(recordId: string)
        +UpdateGoogleTokenStatusDeleted(token, message, sendNotify)
        +UpdateMicrosoftTokenStatusDeleted(token, message, sendNotify)
        +GetFromCache(cache, id)
        +UpdateToCache(cache, token)
    }
    
    class RecordDBPractice {
        <<abstract>>
        +GetAsync()
        +AddAsync()
        +UpdateAsync()
        +DeleteManyAsync()
    }
    
    ApplicationUserAccessTokensDB --|> RecordDBPractice
```

---

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

### 8.3 Provider Routing Logic

```javascript
// Frontend routing on login
if (currentUser.emailStatus?.provider === 0) {
    navigate('/admin/emails/google');  // Gmail provider
} else {
    navigate('/admin/emails');          // Outlook provider
}
```

### 8.4 Backend Provider Resolution

```csharp
// Email Service Controller
if (accessToken.EmailProvider == ApplicationEmailServiceProviders.Outlook) {
    // Route to Microsoft Controller
    var client = _graphSdkHelper.GetAuthenticatedClient(accessToken);
    var res = await MicrosoftController.GetMessages(/* parameters */);
} 
else if (accessToken.EmailProvider == ApplicationEmailServiceProviders.Gmail) {
    // Route to Google Controller
    var controller = new GoogleController(/* dependencies */);
    var res = await controller.GetMessageList(/* parameters */);
}
```

---

## 9. Error Handling & Recovery

### 9.1 Failed Email Queue Recovery

When email sending fails, the system implements an automatic recovery mechanism:

```mermaid
flowchart TD
    A[Email Send Fails] --> B[Check Error Type]
    B -->|Reauthorize Error| C[Store in EmailQueueItemError]
    B -->|Other Error| D[Log Error]
    C --> E[User Re-links Account]
    E --> F[AuthCallback Triggered]
    F --> G[Query Failed Emails]
    G --> H{Found Failed Emails?}
    H -->|Yes| I[Filter: Last 15 Days<br/>Provider Match<br/>Reauthorize Type]
    H -->|No| J[Continue Normal Flow]
    I --> K[Convert to QueueItems]
    K --> L[Re-insert to EmailQueueItem]
    L --> M[Retry Sending]
```

### 9.2 Token Status Notification Flow

When token becomes invalid, the system notifies the user:

```mermaid
sequenceDiagram
    participant System
    participant DB
    participant EmailSender
    participant User
    
    System->>DB: UpdateTokenStatusDeleted(token, message)
    DB->>DB: Set Status = Deleted
    DB->>DB: Set LastError = message
    DB->>DB: SetEmailStatus(NeedApproval)
    
    alt sendNotifyToUser = true
        DB->>EmailSender: SendEmailAsync()
        Note over EmailSender: Email Template:<br/>- Account unlinked notification<br/>- Re-link instructions<br/>- Profile link button
        EmailSender->>User: Notification Email
    end
    
    DB-->>System: Success/Failure
```

### 9.3 Email Template Structure

The notification email includes:
- **Subject**: "Account unlinked"
- **Content**:
  - Logo and branded header
  - Personalized greeting
  - Explanation of why account was unlinked
  - Call-to-action button with profile link
  - Instructions for re-linking
  - Team signature

---

## 10. Key Design Patterns

### 10.1 Service Separation

The system employs clear separation of concerns:

```mermaid
graph TB
    subgraph "Acting Office APIs"
        A1[User Management]
        A2[Account Linking]
        A3[Scheduled Communications]
        A4[Token Status Management]
    end
    
    subgraph "Acting Office Email Service"
        B1[Real-time Email Operations]
        B2[Provider Routing]
        B3[Email CRUD Operations]
    end
    
    A2 -.->|Link/Unlink Operations| B1
    B1 -.->|Check Token Status| A4
```

### 10.2 Provider Abstraction

Both Google and Microsoft providers are accessed through a unified interface:

```typescript
interface IEmailProvider {
    GetMessages(/* parameters */): Promise<PagedData<EmailsListItem>>;
    SendMessage(/* parameters */): Promise<SendResult>;
    DeleteMessage(id: string): Promise<boolean>;
    // ... other common operations
}
```

### 10.3 Caching Strategy

```mermaid
flowchart LR
    A[Request Token] --> B{Check Cache}
    B -->|Hit| C[Return Cached Token]
    B -->|Miss| D[Query Database]
    D --> E[Store in Cache<br/>TTL: 2 hours]
    E --> F[Return Token]
```

---

## 11. Security Considerations

### 11.1 Authentication Requirements

- All email operations require authenticated users with roles: ADMIN, MANAGER, or STAFF
- Link/Unlink operations support both authenticated and unauthenticated scenarios
- Tokens are securely stored with encryption

### 11.2 Token Security

```mermaid
graph TD
    A[Token Storage] --> B[Encrypted at Rest]
    A --> C[Cached with TTL]
    A --> D[Automatic Expiration]
    D --> E[Refresh Mechanism]
    E --> F{Refresh Success?}
    F -->|No| G[User Re-authentication Required]
    F -->|Yes| H[Continue Operations]
```

### 11.3 Subscription Management

For Microsoft Outlook:
- Subscriptions are created for both Inbox and Sent folders
- Subscriptions are automatically renewed before expiration
- On unlink, all subscriptions are properly deleted
- Failed subscription deletions are handled gracefully (try-catch)

---

## 12. Best Practices

### 12.1 Error Recovery

1. **Automatic Retry**: Failed emails are automatically queued for retry
2. **User Notification**: Users are notified when manual intervention is required
3. **Graceful Degradation**: System continues functioning even if one provider fails

### 12.2 Token Management

1. **Proactive Refresh**: Tokens are refreshed 480 seconds (8 minutes) before expiration
2. **Status Tracking**: Token status is maintained for monitoring
3. **Cache Optimization**: Frequently accessed tokens are cached for 2 hours

### 12.3 Database Operations

1. **Practice Isolation**: All operations are scoped to practice ID
2. **Efficient Queries**: Indexes on userId and emailProvider
3. **Batch Operations**: Multiple subscriptions handled in loops

---

## Appendix A: Enumerations

### ApplicationEmailServiceProviders
```csharp
enum ApplicationEmailServiceProviders {
    Gmail = 0,
    Outlook = 1
}
```

### UserAccessTokenStatus
```csharp
enum UserAccessTokenStatus {
    Active,
    Expired,
    Deleted,
    Error
}
```

### ApplicationUserEmailStatus
```csharp
enum ApplicationUserEmailStatus {
    Connected,
    NotConnected,
    NeedApproval
}
```

---

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