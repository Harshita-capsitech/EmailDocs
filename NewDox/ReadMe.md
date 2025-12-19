# Acting Office Email System

## Overview
The Acting Office Email System is a multi-provider email management platform that integrates with Microsoft Outlook and Google Gmail. The system provides comprehensive email operations including real-time management, scheduled delivery, AI-powered features, and seamless provider switching.

## System Architecture

### Backend Services
- **Acting Office APIs**: Handles authentication, provider linking/unlinking, and Communication Email module operations
- **Acting Office Email Service**: Manages real-time email operations for the Emails module
- **Acting Office Sync Service**: Synchronizes emails from providers to local database for offline access
- **Acting Office Schedule Service**: Manages database-driven email scheduling and automatic delivery
- **Acting Office AI Service (ConvoMail)**: Powers AI features like translation, summarization, and tone analysis

### Email Providers
- **Microsoft Outlook**: Integration via Microsoft Graph API with Azure AD authentication
- **Google Gmail**: Integration via Gmail API with OAuth 2.0 authentication

## Modules

### 1. Emails Module
Real-time email management with instant access to inbox, sent items, and drafts. Provides standard email operations including compose, send, organize, search, and filter across both Outlook and Gmail providers.

📄 [View Detailed Documentation →](./Inbox.md)

### 2. Communication Email Module
Advanced email management with all features of the Emails Module plus scheduling capabilities. Schedule emails for future delivery with automatic retry logic and delivery tracking.

📄 [View Detailed Documentation →](./Communication.md)

### 3. Sync Mail Module
Database synchronization system that keeps the local database in sync with email provider inboxes. Provides offline access, faster retrieval, and reduces dependency on provider APIs by maintaining a local copy of emails.

**Key Capabilities:**
- **Automatic Sync**: Periodic synchronization with Outlook/Gmail
- **Incremental Updates**: Only fetches new or modified emails
- **Offline Access**: View emails from local database when provider is unavailable
- **Reduced API Calls**: Minimizes direct provider API dependencies
- **Bi-directional Sync**: Local changes reflected back to provider

### 4. Schedule Mail Module
Independent email scheduling system that stores scheduled emails in the local database instead of relying on provider APIs. Provides complete control over email delivery timing with automatic sending at scheduled times.

**Key Capabilities:**
- **Database Storage**: Scheduled emails stored in local database
- **Independent Execution**: Not dependent on provider API scheduling features
- **Flexible Scheduling**: Schedule emails for any future date/time
- **Automatic Delivery**: Background service sends emails at scheduled time
- **Retry Logic**: Automatic retry on delivery failures
- **Status Tracking**: Monitor scheduled, sent, and failed emails

### 5. AI Features
AI-powered email enhancements including translation (any language → English), content summarization, tone analysis, auto-complete suggestions, contextual reply generation, and automatic response drafting.

**Available Features:**
- **Translate**: Convert email content to English from any language
- **Summarize**: Generate concise or detailed email summaries
- **Tone Analysis**: Identify email tone (professional, neutral, urgent, etc.)
- **Auto-Complete**: Smart suggestions while composing
- **Reply Suggestions**: Context-aware reply options
- **Draft Generation**: Automatic response creation based on email intent

## Key Features

### Authentication & Provider Management
- OAuth 2.0 / Azure AD authentication
- Support for both Outlook and Gmail
- Easy account linking/unlinking
- Automatic provider detection and routing

### Email Operations
- View inbox, sent items, and drafts
- Send and receive emails
- Organize with folders/labels
- Advanced search and filtering
- Attachment management
- Draft saving and editing

### Sync Mail (Database Synchronization)
- Automatic email synchronization from provider to database
- Incremental sync for new/modified emails only
- Offline email access from local database
- Reduced API dependency and faster retrieval
- Bi-directional sync capability
- Configurable sync intervals

### Schedule Mail (Independent Scheduling)
- Database-driven email scheduling
- Schedule emails for future delivery without provider API
- Automatic background sending at scheduled time
- Automatic retry on failures
- Delivery status tracking
- Queue management with retry logic

### Token Management
- Automatic token refresh
- Proactive expiration handling
- User notification for re-authorization
- Seamless re-authentication flow

## Common Use Cases

### 1. Link Email Provider
Connect your Microsoft Outlook or Google Gmail account to enable email features.

**Steps**: Login → Navigate to Profile → Click Link Button → Authorize Provider → Success

### 2. Send Immediate Email
Compose and send emails instantly through either module.

**Available In**: Emails Module, Communication Email Module

### 3. Sync Emails to Database
Automatically synchronize emails from provider inbox to local database for faster access and offline availability.

**Available In**: Sync Mail Module
**Benefits**: Faster retrieval, offline access, reduced API calls

### 4. Schedule Email for Later
Plan email delivery for a specific future date and time using database storage (independent of provider API).

**Available In**: Schedule Mail Module, Communication Email Module
**Storage**: Local database with automatic background delivery

### 5. AI-Enhanced Email Composition
Use AI features to translate, summarize, or generate contextual responses.

**Available In**: Emails Module (AI integration)

### 6. Switch Email Provider
Unlink current provider and link a different one (Outlook ↔ Gmail).

**Steps**: Unlink Current Account → Re-authorize → Link New Provider

## Provider Comparison

| Feature | Microsoft Outlook | Google Gmail |
|---------|------------------|--------------|
| **Provider Code** | 1 | 0 |
| **Authentication** | Azure AD (OpenID Connect) | OAuth 2.0 |
| **API** | Microsoft Graph API | Gmail API |
| **Notifications** | Webhook Subscriptions | Push Notifications |
| **Supported Modules** | ✅ Emails<br>✅ Communication Email<br>✅ AI Features | ✅ Emails<br>✅ Communication Email<br>✅ AI Features |

## Module Comparison

| Feature | Emails Module | Communication Module | Sync Mail Module | Schedule Mail Module |
|---------|---------------|---------------------|------------------|---------------------|
| **Backend Service** | Email Orchestration Service | Acting Office APIs | Acting Office APIs | Acting Office APIs |
| **Data Storage** | Provider API only | Database + Provider API | Local Database | Local Database |
| **View Inbox** | ✅ Real-time from provider | ✅ Real-time from provider | ✅ From local database | ❌ |
| **Send Email** | ✅ Immediate | ✅ Immediate | ❌ | ✅ Scheduled only |
| **Organize Folders** | ✅ | ✅ | ✅ Synced folders | ❌ |
| **Search & Filter** | ✅ Provider-based | ✅ Provider-based | ✅ Database-based | ✅ Schedule queue |
| **Draft Management** | ✅ | ✅ | ✅ Synced drafts | ❌ |
| **Schedule Email** | ❌ | ✅ | ❌ | ✅ Primary feature |
| **Offline Access** | ❌ | ❌ | ✅ | ✅ Schedule data |
| **Sync Capability** | ❌ | ❌ | ✅ Bi-directional | ❌ |
| **API Dependency** | High | High | Low (after sync) | Low |
| **Delivery Tracking** | Basic | ✅ | ❌ | ✅ Comprehensive |
| **Automatic Retry** | ❌ | ✅ | ❌ | ✅ |
| **AI Features** | ✅ | ✅ | ✅ | ❌ |

## Quick Reference

### Authentication Endpoints
- **Link Outlook**: `/Addons/Microsoft/Link`
- **Unlink Outlook**: `/Addons/Microsoft/Unlink`
- **Link Gmail**: `/Addons/Google/Link`
- **Unlink Gmail**: `/Addons/Google/Unlink`

### Email Operations (Emails Module)
- **Inbox**: `/Emails/Inbox`
- **Send**: `/Emails/Send`
- **Draft**: `/Emails/Draft`
- **Search**: `/Emails/Search`

### Email Operations (Communication Email Module)
- **Send Email**: `/SendEmail`
- **Schedule Email**: `/SaveSchedule`
- **Get Scheduled**: `/GetScheduleMails`
- **Get Drafts**: `/GetDraftMails`

### Sync Mail Operations
- **Trigger Sync**: `/Sync/Inbox`
- **Sync Status**: `/Sync/Status`
- **Get Synced Emails**: `/Sync/Emails`
- **Sync Settings**: `/Sync/Settings`

### Schedule Mail Operations
- **Create Schedule**: `/Schedule/Create`
- **Get Schedules**: `/Schedule/List`
- **Update Schedule**: `/Schedule/{id}`
- **Delete Schedule**: `/Schedule/{id}/Delete`
- **Send Now**: `/Schedule/{id}/SendNow`

### AI Features
- **Translate**: `/translate`
- **Summarize**: `/summarize`
- **Tone Analysis**: `/tone`
- **Draft Email**: `/draftMail`
- **Reply Suggestions**: `/replies`
- **Auto-Complete**: `/completion`

## Business Rules

### Provider Linking
- Only one provider can be linked at a time
- Account must be re-linked if credentials expire
- Unlinking removes all subscriptions and tokens
- Failed emails from last 15 days are re-queued on re-linking

### Sync Mail
- Automatic sync runs at configurable intervals (default: every 5 minutes)
- Incremental sync only fetches new/modified emails since last sync
- Local database maintains copy of emails for offline access
- Sync conflicts resolved with provider data as source of truth
- Failed sync attempts retry automatically
- Sync history maintained for audit purposes

### Schedule Mail (Database-Driven)
- Scheduled emails stored in local database, not provider API
- Background service checks queue every minute for due emails
- Automatic sending when scheduled time is reached
- Maximum retry attempts: 3 (configurable based on error type)
- Automatic retry after account re-linking for auth failures
- User notified on permanent failures after max retries
- Draft emails can be converted to scheduled emails
- Scheduled emails can be edited before send time

### Email Scheduling (Communication Module)
- Scheduled emails stored in queue with retry logic
- Maximum retry attempts based on error type
- Automatic retry after account re-linking
- User notified on permanent failures

### Token Management
- Tokens automatically refreshed before expiration
- 8-minute buffer for Google token refresh
- User notified when re-authorization required
- Failed operations queued for retry after re-linking

### AI Features
- Translation defaults to English output
- Tone analysis covers professional, neutral, urgent tones
- Auto-complete provides context-aware suggestions
- Reply generation based on email intent and context

## System Characteristics

### Scalability
- Practice-level data isolation
- Distributed caching for performance
- Queue-based scheduling architecture
- Horizontal scaling capability
- Database-driven sync reduces API load
- Efficient incremental sync mechanism

### Reliability
- Automatic token refresh
- Failed email retry mechanism (15-day retention)
- Comprehensive error handling
- Delivery status tracking
- Offline email access via Sync Mail
- Independent scheduling with database storage

### Security
- OAuth 2.0 / OpenID Connect authentication
- Encrypted token storage
- Role-based access control
- Audit trail for operations
- Secure local database storage
- Data encryption at rest and in transit

### Performance
- Reduced API dependency through local database
- Faster email retrieval from synced data
- Optimized incremental sync
- Background scheduling service
- Efficient queue processing

## Best Practices

✅ **Regular Token Refresh**: System handles automatic token refresh - no user action needed

✅ **Re-link Promptly**: Re-authorize account immediately when notified to ensure email continuity

✅ **Use Sync Mail for Performance**: Enable sync for faster email access and reduced API calls

✅ **Monitor Sync Status**: Regularly check sync status to ensure database is up-to-date

✅ **Use Schedule Mail for Reliability**: Database-driven scheduling provides better control and tracking

✅ **Configure Sync Intervals**: Adjust sync frequency based on email volume and requirements

✅ **Leverage Offline Access**: Use synced emails when provider APIs are unavailable

✅ **Use Scheduling Wisely**: Use Schedule Mail Module for time-sensitive emails requiring delivery tracking

✅ **Leverage AI Features**: Utilize AI tools for translation, summarization, and professional communication

✅ **Monitor Scheduled Emails**: Regularly check scheduled email status in Schedule Mail Module

✅ **Choose Right Module**: Use Emails Module for immediate operations, Schedule Mail for future delivery, Sync Mail for offline access

## Troubleshooting

### Common Issues

**Issue**: Cannot access emails
- **Solution**: Check if account is linked. If status shows "NeedApproval", re-link your account.

**Issue**: Scheduled email not sent
- **Solution**: Verify account link status. Check Schedule Mail queue. System automatically retries after re-linking.

**Issue**: Sync not working
- **Solution**: Check sync status and last sync time. Verify account link. Manually trigger sync if needed.

**Issue**: Emails not appearing in synced database
- **Solution**: Check sync service status. Verify sync settings. Review sync error logs.

**Issue**: Scheduled email stuck in queue
- **Solution**: Check background service status. Verify scheduled time. Check for authentication errors.

**Issue**: Token expired error
- **Solution**: System attempts automatic refresh. If it fails, you'll be notified to re-link.

**Issue**: Provider switch not working
- **Solution**: Completely unlink current provider before linking new one.

**Issue**: Slow email retrieval
- **Solution**: Enable Sync Mail for faster access. Check if sync is up-to-date.

## Process Documentation

- [Email Module Documentation](./emailsection.md) - Detailed Email Module workflows
- [Communication Email Module Documentation](./communication-email-section.md) - Scheduling and tracking processes
- [Technical Documentation](./acting-office-email-system-technical-docs.md) - Complete technical reference

## Entity Documentation

Key data entities:
- **ApplicationUserAccessTokens**: User authentication tokens
- **UserMicrosoftAccessToken**: Microsoft-specific credentials
- **UserGoogleAccessToken**: Google-specific credentials
- **EmailQueueItem**: Scheduled emails queue (Communication Module)
- **EmailQueueItemError**: Failed delivery tracking
- **SyncedEmail**: Synchronized emails stored in local database
- **SyncStatus**: Email synchronization status and history
- **ScheduledEmailItem**: Database-stored scheduled emails (Schedule Mail Module)
- **EmailDeliveryLog**: Email delivery tracking and audit trail

## API Documentation

### Swagger Endpoints
- **Acting Office APIs**: [https://apiuat.actingoffice.com/api-docs](https://apiuat.actingoffice.com/api-docs)
- **Email Service**: [https://emails.servicesuat.actingoffice.com/swagger](https://emails.servicesuat.actingoffice.com/swagger)
- **AI Service**: [https://ai.servicesuat.actingoffice.com/swagger](https://ai.servicesuat.actingoffice.com/swagger)

## See Also

- [Microsoft Graph API Documentation](https://docs.microsoft.com/graph) - Outlook integration reference
- [Gmail API Documentation](https://developers.google.com/gmail/api) - Gmail integration reference
- [OAuth 2.0 Documentation](https://oauth.net/2/) - Authentication protocol
- [Azure AD Documentation](https://docs.microsoft.com/azure/active-directory) - Microsoft identity platform

---

*Document Version: 1.0*  
*Last Updated: December 2025*  
*Acting Office Email System - Overview & Quick Start*
