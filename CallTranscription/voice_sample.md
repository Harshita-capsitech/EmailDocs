# Voice Recording & Verification Module

---

## 1. Overview

The Voice Recording & Verification module provides a secure mechanism for verifying a user’s identity using voice biometrics.
It is integrated into the Profile section and is designed with clear separation of concerns between UI, AI verification, and backend persistence.

### Goals
- Enable voice-based identity verification
- Prevent storage of raw biometric data in core systems
- Provide clear auditability of verification status
- Ensure extensibility for future biometric workflows

### Non-Goals
- Long-term storage of voice samples
- Voice authentication at login time
- Multi-factor biometric comparison

---

## 2. Architecture Summary

The module is composed of three logical layers:

1. **Presentation Layer**
   - VoiceRecordingModal
   - Profile UI

2. **Verification Layer**
   - AI audio verification service (`/sample`)

3. **Persistence Layer**
   - AuthController.UploadAudio
   - MongoDB (ApplicationUser collection)

This layered approach ensures that biometric data does not leak into core authentication services.

---

flowchart LR
    U[User] -->|Speaks sample text| B[Browser / UI]
    
    B -->|Raw audio stream| VR[Voice Recording Module]
    VR -->|Verified audio (WAV)| PS[ProfileService]

    PS -->|FormData (audio + metadata)| AI[AI Verification Service<br/>/sample]
    AI -->|Verification status| PS

    PS -->|Verification flag| AC[AuthController<br/>/Auth/UploadAudio]
    AC -->|Persist voice status| DB[(MongoDB<br/>ApplicationUser)]

    DB -->|Voice verification state| UI[Profile UI]


## 4. Process Flow

```mermaid
sequenceDiagram
    participant U as User
    participant V as VoiceRecordingModal
    participant PS as ProfileService
    participant AI as AI Verification API
    participant AC as AuthController
    participant DB as MongoDB

    U->>V: Start recording
    V->>V: Request mic permission
    V->>V: Capture audio chunks
    U->>V: Stop recording / auto-stop
    V->>V: Convert WebM → WAV
    V->>PS: checkAudio(FormData)
    PS->>AI: POST /sample
    AI-->>PS: { status, message }
    PS->>AC: POST /Auth/UploadAudio (true/false)
    AC->>DB: Update user.voice
    DB-->>AC: ModifiedCount
    AC-->>PS: ApiResponse
    PS-->>V: Verification outcome
    V-->>U: Success / failure UI
```

---

## 5. ER Diagram

```mermaid
erDiagram
    ApplicationUser ||--o| ApplicationUserVoiceStatus : has

    ApplicationUser {
        string Id
        int PracticeId
        ApplicationUserVoiceStatus Voice
    }

    ApplicationUserVoiceStatus {
        bool Recorded
        datetime On
    }
```

---

## 6. Entity Definition

### ApplicationUserVoiceStatus

Represents the verification state of a user’s voice identity.

Fields:
- **Recorded**: Indicates whether verification succeeded
- **On**: Timestamp of last successful verification

This structure is intentionally minimal to reduce biometric footprint.

---

## 7. Authentication & APIs

### Frontend APIs (ProfileService)

#### checkAudio
- Endpoint: `/sample`
- Method: POST
- Payload: multipart/form-data
- Content:
  - WAV audio file
  - Agent metadata (id, name)
- Responsibility:
  - AI-based voice verification

#### uploadAudio
- Endpoint: `/Auth/UploadAudio`
- Method: POST
- Payload: boolean
- Responsibility:
  - Persist verification result

---

### Backend API (AuthController)

#### POST /Auth/UploadAudio

Behavior:
1. Validates authenticated user
2. Filters MongoDB document using:
   - PracticeId
   - UserId
3. Updates:
   - user.voice.recorded
   - user.voice.on

No raw audio is stored or processed at this layer.

---

## 8. Security Considerations

- Raw audio never reaches AuthController
- AI service is isolated and accessed via OIDC
- Voice status is boolean, not biometric
- MongoDB stores metadata only
- Browser permissions strictly enforced

---

## 9. Error Handling & Recovery

### Frontend
- Permission denial handling
- Unsupported browser detection
- Silent audio detection
- Retry and re-record support

### Backend
- MongoDB update validation
- Graceful failure on no modification
- Standard ApiResponse error propagation

---

## 10. Testing Guide

### Frontend Testing
- Microphone permission flows
- Auto-stop after 60 seconds
- Success and failure UI paths
- Re-record functionality

### API Testing
POST /Auth/UploadAudio
Body: true / false

Expected:
- status = true
- result contains updated voice object

### Database Validation
Query ApplicationUser.Voice fields directly in MongoDB.

---

## 11. Performance Notes

- Audio processing done client-side
- Small payloads to backend
- No blocking operations on profile load
- Minimal database write footprint

---

## 12. Future Enhancements

- Voice confidence score storage
- Multiple language sample support
- Expiry-based re-verification
- Admin audit view

---

## 13. References

- VoiceRecordingModal.tsx
- Profile.tsx
- ProfileService.tsx
- AuthController.cs

---

## 14. Version & Change Log

### v1.0.0
- Initial voice verification
- AI integration
- MongoDB persistence

### v1.1.0
- WAV conversion
- Auto-stop recording
- Improved error handling

### v1.2.0
- UI sync improvements
- Re-record flow
- Background user refresh

---

End of Document
