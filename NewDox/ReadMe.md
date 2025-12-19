# Acting Office Email System

## Overview
The Acting Office Email System is a multi-provider email management platform that integrates with Microsoft Outlook and Google Gmail. The system provides comprehensive email operations including real-time management, scheduled delivery, AI-powered features, and seamless provider switching.

## System Architecture

### Backend Services
- **Acting Office APIs**: Handles authentication, provider linking/unlinking, and Communication Email module operations
- **Acting Office Email Service**: Manages real-time email operations for the Emails module
- **Acting Office AI Service (ConvoMail)**: Powers AI features like translation, summarization, and tone analysis

### Email Providers
- **Microsoft Outlook**: Integration via Microsoft Graph API with Azure AD authentication
- **Google Gmail**: Integration via Gmail API with OAuth 2.0 authentication

## Modules

### 1. Emails Module
Real-time email management with instant access to inbox, sent items, and drafts. Provides standard email operations including compose, send, organize, search, and filter across both Outlook and Gmail providers.

📄 [View Detailed Documentation →](./emailsection.md)

### 2. Communication Email Module
Advanced email management with all features of the Emails Module plus scheduling capabilities. Schedule emails for future delivery with automatic retry logic and delivery tracking.

📄 [View Detailed Documentation →](./Communication.md)

### 3. AI Features
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

### Scheduling (Communication Module Only)
- Schedule emails for future delivery
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

### 3. Schedule Email for Later
Plan email delivery for a specific future date and time with automatic retry.

**Available In**: Communication Email Module only

### 4. AI-Enhanced Email Composition
Use AI features to translate, summarize, or generate contextual responses.

**Available In**: Emails Module (AI integration)

### 5. Switch Email Provider
Unlink current provider and link a different one (Outlook ↔ Gmail).

**Steps**: Unlink Current Account → Re-authorize → Link New Provider



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

### Reliability
- Automatic token refresh
- Failed email retry mechanism (15-day retention)
- Comprehensive error handling
- Delivery status tracking

### Security
- OAuth 2.0 / OpenID Connect authentication
- Encrypted token storage
- Role-based access control
- Audit trail for operations

## Best Practices

✅ **Regular Token Refresh**: System handles automatic token refresh - no user action needed

✅ **Re-link Promptly**: Re-authorize account immediately when notified to ensure email continuity

✅ **Use Scheduling Wisely**: Use Communication Email Module for time-sensitive emails requiring delivery tracking

✅ **Leverage AI Features**: Utilize AI tools for translation, summarization, and professional communication

✅ **Monitor Scheduled Emails**: Regularly check scheduled email status in Communication Module

✅ **Choose Right Module**: Use Emails Module for immediate operations, Communication Module for scheduling

## Troubleshooting

### Common Issues

**Issue**: Cannot access emails
- **Solution**: Check if account is linked. If status shows "NeedApproval", re-link your account.

**Issue**: Scheduled email not sent
- **Solution**: Verify account link status. System automatically retries after re-linking.

**Issue**: Token expired error
- **Solution**: System attempts automatic refresh. If it fails, you'll be notified to re-link.

**Issue**: Provider switch not working
- **Solution**: Completely unlink current provider before linking new one.

## Process Documentation

- [Email Module Documentation](./emailsection.md) - Detailed Email Module workflows
- [Communication Email Module Documentation](./communication-email-section.md) - Scheduling and tracking processes
- [Technical Documentation](./acting-office-email-system-technical-docs.md) - Complete technical reference

## Entity Documentation

Key data entities:
- **ApplicationUserAccessTokens**: User authentication tokens
- **UserMicrosoftAccessToken**: Microsoft-specific credentials
- **UserGoogleAccessToken**: Google-specific credentials
- **EmailQueueItem**: Scheduled emails queue
- **EmailQueueItemError**: Failed delivery tracking

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