# Voice Recording & Verification Module

## Overview
The Voice Recording & Verification module enables voice-based identity verification from the Profile section of the application.
It integrates browser-side audio recording, AI-powered verification, and backend persistence of verification status.

## DFD (Data Flow Diagram)
```mermaid
flowchart LR
    User --> Browser
    Browser --> VoiceRecordingModal
    VoiceRecordingModal --> AudioProcessor
    AudioProcessor --> AIService["/sample"]
    AIService --> ProfileService
    ProfileService --> AuthController["/Auth/UploadAudio"]
    AuthController --> MongoDB
    MongoDB --> ProfileUI
```

## Process Flow
```mermaid
sequenceDiagram
    participant User
    participant VoiceRecordingModal
    participant ProfileService
    participant AIService
    participant AuthController
    participant MongoDB

    User->>VoiceRecordingModal: Start Recording
    VoiceRecordingModal->>VoiceRecordingModal: Capture Audio
    User->>VoiceRecordingModal: Stop Recording
    VoiceRecordingModal->>ProfileService: checkAudio()
    ProfileService->>AIService: POST /sample
    AIService-->>ProfileService: Verification Result
    ProfileService->>AuthController: POST /Auth/UploadAudio
    AuthController->>MongoDB: Update Voice Status
```

## ER Diagram
```mermaid
erDiagram
    ApplicationUser ||--o| ApplicationUserVoiceStatus : has
```

## Entity Definition
ApplicationUserVoiceStatus:
- recorded: boolean
- on: datetime

## Authentication / APIs
- POST /sample (AI verification)
- POST /Auth/UploadAudio (persist verification status)

## Testing Guide
- Manual UI testing via Profile screen
- API testing via Postman

## References
- VoiceRecordingModal.tsx
- Profile.tsx
- ProfileService.tsx
- AuthController.cs

## Version and Change Log
v1.0.0 Initial implementation
