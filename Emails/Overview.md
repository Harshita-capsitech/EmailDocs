# **Email Module Documentation**

# **Overview**

This **Email Module** integrates with two major email service providers—**Outlook** and **Gmail**—and provides a comprehensive solution for managing emails, drafts, and attachments within your system. The integration is designed to dynamically select the email service provider based on the practice ID, allowing seamless and personalized email management for users. Whether you're using Outlook or Gmail, the system ensures smooth interaction with both providers while leveraging advanced features to enhance communication.The core functionality of this module spans essential email management tasks, from composing and sending emails to organizing messages into folders. Users can manage their inbox, sent items, drafts, and other custom folders, offering full flexibility in how emails are organized. Additionally, the module supports managing email attachments, including uploading and downloading files, ensuring that users can easily exchange documents and multimedia.

   One of the standout features of this email module is its **AI-driven capabilities**. These intelligent features, including automatic response generation, email summarization, and translation, enhance user productivity and streamline email communication. The AI features make it easier for users to handle routine email responses, summarize long emails for quicker reading, and even translate emails into their preferred language.


The module provides several key features, such as:

- **Compose Email** with options to set email importance (Low, Normal, High).
- **Manage Folders**: Inbox, Sent, Drafts, Archive, etc.
- **AI Features**: Translate, Summarize, Short Replies.
- **Attachment Handling**: Upload, Download, and Delete Attachments.
- **Search and Filters**: Apply filters for read/unread emails, importance, and more.
- **Conversation History**: Maintain email threads for ongoing conversations.

---

## DFD (Data Flow Diagram)

Here is the **Data Flow Diagram** illustrating the interaction between the user, system components, and the email service providers.

```mermaid
flowchart TD
    %% External Entity
    User[(User)]
    
    %% Authentication
    P1((Authenticate User))
    AuthService[Authentication Auth Service]
    
    %% Core System
    UI[User Interface]
    Backend["Backend System"]
    DB[(Database)]
    
    %% Email Providers
    EmailProvider{Email Provider}
    Outlook["Outlook API"]
    Gmail["Gmail API"]
    
    %% AI
    AI["AI Service API"]
    
    %% Flow
    User edge1@--> P1
    P1 edge2@--> AuthService
    AuthService edge3@--> P1
    P1 edge4@-->|Authenticated| UI
    UI edge5@--> Backend
    Backend edge6@--> DB
    DB edge7@--> Backend
    Backend edge8@--> EmailProvider
    EmailProvider edge9@-->|Outlook| Outlook
    EmailProvider edge10@-->|Gmail| Gmail
    Outlook edge11@--> Backend
    Gmail edge12@--> Backend
    Backend edge13@--> UI
    UI edge14@--> AI
    AI edge15@--> UI
    
    %% Animations
    edge1@{ animation: fast }
    edge2@{ animation: fast }
    edge3@{ animation: fast }
    edge4@{ animation: fast }
    edge5@{ animation: fast }
    edge6@{ animation: fast }
    edge7@{ animation: fast }
    edge8@{ animation: fast }
    edge9@{ animation: fast }
    edge10@{ animation: fast }
    edge11@{ animation: fast }
    edge12@{ animation: fast }
    edge13@{ animation: fast }
    edge14@{ animation: fast }
    edge15@{ animation: fast }
    
    %% Links
    click Backend "https://apiuat.actingoffice.com/api-docs/index.html" "Backend API Documentation"
    click AI "https://ai.servicesuat.actingoffice.com/swagger/?urls.primaryName=Acting+Office+-+Convomail#/default/suggest_suggest_post" "AI Service API Documentation"

```

---

## **Process Flow**

### **Description:**
The process flow outlines the step-by-step operations within the Email Module. It provides a seamless user experience, from selecting an email provider to composing and sending emails, managing drafts, and leveraging AI features to enhance communication.

1. **Email Provider Selection**:
   - The system automatically determines the email service provider (either **Outlook** or **Gmail**) based on the **practice ID**. The selected email provider enables users to access their respective email services, manage emails, and interact with system features such as sending and receiving emails.

2. **Compose Email**:
   - The user navigates to the email composition screen where they can:
     - Set the **email importance** to **Low**, **Normal**, or **High**, ensuring that recipients understand the priority of the email.
     - Write the body content of the email, including formatting options (e.g., bold, italics, bullet points, etc.).
     - **Attach files** by selecting them from the local file system or by using a drag-and-drop interface.
     - Select recipients, including **To**, **CC**, and **BCC**.

3. **Email Sending**:
   - Once the email is composed, the system sends it to the selected email provider’s API (either **Outlook** or **Gmail**), where it is processed and delivered to the recipients. The email is sent with all the composed content, attachments, and any additional metadata such as **importance** and **attachments**.

4. **Drafts & Sent Items**:
   - Emails that are not sent immediately are stored as **Drafts** within the **Drafts folder**. The user can access and edit the draft later before sending it. Once an email is successfully sent, it is moved to the **Sent Items** folder, allowing the user to track emails that have been sent.

5. **AI Features**:
   - When viewing an email, the user can take advantage of AI-based features that assist in understanding and responding to the message:
     - **Auto Response**: AI suggests a suitable reply based on the content of the received email, saving time for routine responses.
     - **Translation**: If the email is written in a foreign language, the system can translate the email body into English (or another language) to facilitate comprehension.
     - **Email Summary**: AI provides a brief summary of the email’s content, enabling users to understand the main points without reading the entire body.
     - **Short Replies**: Based on the context of the email, AI suggests short, contextual replies to help users respond quickly.

---

### **Process Flow Diagram**:

```mermaid
flowchart TD
%% ========== User & Authentication ==========
User[(User)]
Login([Login])
AuthProcess((Authenticate User))
AuthService[Authentication Service]
AuthCheck{Authenticated?}

User edge1@--> Login
Login edge2@--> AuthProcess
AuthProcess edge3@--> AuthService
AuthService edge4@--> AuthProcess
AuthProcess edge5@--> AuthCheck

AuthCheck edge6@-- No --> Login

%% ========== Provider Selection ==========
AuthCheck edge7@-- Yes --> Provider{Email Provider}
Provider edge8@-- Outlook --> OutlookRoute[Outlook]
Provider edge9@-- Gmail --> GmailRoute[Gmail]

%% ========== Mailbox & Folders ==========
OutlookRoute edge10@--> Mailbox[Mailbox]
GmailRoute edge11@--> Mailbox

Mailbox edge12@--> Folders[Folders<br/>Inbox, Sent, Drafts, Trash]

%% ========== Compose Email ==========
Folders edge13@--> Compose[Compose Email]
Compose edge14@--> Editor[Content Editor]
Editor edge15@--> Recipients[Recipients & Priority]
Recipients edge16@--> Attachments[Attachments]
Attachments edge17@--> SendAction[Send / Save Draft]
SendAction edge18@--> Folders

%% ========== Read Emails ==========
Folders edge19@--> EmailList[Email List]
EmailList edge20@--> EmailDetail[Email Detail]

%% ========== Email Actions ==========
EmailDetail edge21@--> Actions[Email Actions]
Actions edge22@--> Reply[Reply / Reply All / Forward]
Actions edge23@--> Delete[Delete / Move]

Reply edge24@--> Editor
Delete edge25@--> Folders

%% ========== AI Features ==========
EmailDetail edge26@--> AI[AI Assistance]
AI edge27@--> Summary[Summarize]
AI edge28@--> Translate[Translate]
AI edge29@--> SmartReply[Smart Reply]

%% ========== Animations ==========
edge1@{ animation: fast }
edge2@{ animation: fast }
edge3@{ animation: fast }
edge4@{ animation: fast }
edge5@{ animation: fast }
edge6@{ animation: fast }
edge7@{ animation: fast }
edge8@{ animation: fast }
edge9@{ animation: fast }
edge10@{ animation: fast }
edge11@{ animation: fast }
edge12@{ animation: fast }
edge13@{ animation: fast }
edge14@{ animation: fast }
edge15@{ animation: fast }
edge16@{ animation: fast }
edge17@{ animation: fast }
edge18@{ animation: fast }
edge19@{ animation: fast }
edge20@{ animation: fast }
edge21@{ animation: fast }
edge22@{ animation: fast }
edge23@{ animation: fast }
edge24@{ animation: fast }
edge25@{ animation: fast }
edge26@{ animation: fast }
edge27@{ animation: fast }
edge28@{ animation: fast }
edge29@{ animation: fast }



```
---



## ER Diagram

Here is the **Entity Relationship Diagram (ERD)** that maps out the relationships between core entities in the email system:


```mermaid
erDiagram
    USER {
        string user_id
        string email_address
        string name
        string role
    }
    EMAIL {
        string email_id
        string subject
        string body
        datetime date
        string importance
        boolean is_read
        string folder_id
        string user_id
    }
    ATTACHMENT {
        string attachment_id
        string email_id
        string file_name
        string content_type
        int size
    }
    FOLDER {
        string folder_id
        string display_name
        int message_count
        string user_id
    }

    USER ||--o| EMAIL : has
    EMAIL ||--o| ATTACHMENT : contains
    EMAIL ||--|{ FOLDER : belongs_to
    USER ||--o| FOLDER : has


```

---

## **Entity Definitions**

### **User**
Represents the system user interacting with emails.

- **user_id** (string): Unique identifier for the user
- **email_address** (string): User's email address
- **name** (string): User's full name
- **role** (string): User's role (ADMIN, MANAGER, STAFF)

### **Email**
Represents the email message.

- **email_id** (string): Unique identifier for the email
- **subject** (string): Email subject line
- **body** (text): Email body content
- **date** (datetime): Date and time the email was sent/received
- **importance** (string): Priority level (Low, Normal, High)
- **is_read** (boolean): Whether the email has been read
- **folder_id** (string): Foreign key to the folder
- **user_id** (string): Foreign key to the user

### **Attachment**
Represents an attachment in an email.

- **attachment_id** (string): Unique identifier for the attachment
- **email_id** (string): Foreign key to the email
- **file_name** (string): Name of the attached file
- **content_type** (string): MIME type of the file
- **size** (int): File size in bytes

### **Folder**
Represents email folders.

- **folder_id** (string): Unique identifier for the folder
- **display_name** (string): Name of the folder (Inbox, Sent, Drafts, etc.)
- **message_count** (int): Number of messages in the folder
- **user_id** (string): Foreign key to the user



## **API Endpoints:**

| **Description**                    | **HTTP Method**               | **Endpoint**                                                                 |
|------------------------------------|-------------------------------|-----------------------------------------------------------------------------|
| **Update Email Preferences**       | POST                          | [/admin/emails/preferences](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Get Inbox Emails**               | GET                           | [/Emails/Inbox](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Send Email**                     | POST                          | [/Emails/Message/{id}/send](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Send Email**                     | POST                          | [/Emails/Message/send](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Get Email Details**              | GET                           | [/Emails/Message/{id}](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Upload Attachment**              | POST                          | [/Emails/Message/{messageId}/Attachments](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Upload Attachment In Chunks**    | POST                          | [/Message/{messageId}/Attachments/UploadSession](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Upload Attachment In Chunks**    | PUT                           | [/Attachments/UploadSession/{sessionId}/Chunk](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Upload Attachment In Chunks**    | GET                           | [/Attachments/UploadSession/{sessionId}/Status](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Download Attachment**            | GET                           | [/Message/{messageId}/Attachment/{attachmentId}/Download](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Download All Attachments**       | GET                           | [/Message/{messageId}/Attachment/DownloadAll](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Delete Attachment**              | DELETE                        | [/Message/{messageId}/Attachment/{attachmentId}/Delete](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Parse MIME Message**             | POST                          | [/ParseMIME](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Create Attachment Upload Session**| POST                          | [/Message/{messageId}/Attachments/UploadSession](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Upload Attachment Chunk**        | PUT                           | [/Attachments/UploadSession/{sessionId}/Chunk](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Get Attachment Upload Status**   | GET                           | [/Attachments/UploadSession/{sessionId}/Status](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Move Message**                   | GET                           | [/Move](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Delete Message**                 | GET                           | [/Message/{id}/Delete](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Get Categories**                 | GET                           | [/Categories](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Schedule Email**                 | POST                          | [/ScheduleEmails](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Get Scheduled Emails**           | GET                           | [/ScheduleEmails](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Get Email Folders**              | GET                           | [/Folders](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Create Folder**                  | POST                          | [/Folders](https://emails.servicesuat.actingoffice.com/swagger/index.html) |
| **Get Users**                      | GET                           | [/Users](https://emails.servicesuat.actingoffice.com/swagger/index.html) |





---

## Testing Guide

### **Unit Testing**

- **Test Email Composition**: Validate the ability to create, send, and store email drafts
- **Test AI Features**: Ensure AI features like translation and summarization work as expected
- **Test Attachment Upload**: Verify that attachments can be successfully uploaded and downloaded
- **Test Folder Management**: Validate folder creation, deletion, and email organization

### **Integration Testing**

- **Email Provider API Integration**: Test communication with the Outlook and Gmail APIs
- **Ensure emails are sent and fetched correctly**: Validate email delivery and retrieval
- **Test attachments handling**: Verify attachment upload, download, and deletion across providers
- **Test authentication flows**: Ensure OAuth 2.0 integration works for both providers

### **End-to-End Testing**

- **Full Email Flow**: Test the entire process from composing an email to sending it
- **Include attachment upload**: Verify files are properly attached and sent
- **Test folder management**: Ensure emails are properly organized in folders
- **Test email filters**: Validate filtering by read/unread, importance, attachments
- **Test AI features integration**: Verify translation, summarization, and short replies work end-to-end
- **Test conversation threading**: Ensure email threads are maintained properly

---

## References

- [Outlook API Documentation](https://docs.microsoft.com/en-us/graph/api/resources/mail-api-overview) - Microsoft Graph API
- [Gmail API Documentation](https://developers.google.com/gmail/api) - Google Gmail API
- [OAuth 2.0 Specification](https://oauth.net/2/) - Authentication protocol
- [MIME Types Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types) - File type specifications

---

