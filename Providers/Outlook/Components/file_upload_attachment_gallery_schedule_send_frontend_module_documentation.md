# File Upload, Attachment Gallery & Schedule Send

## Overview
This module provides a **complete email attachment and scheduling experience** used across Compose, Reply, and Forward flows. It includes:
- A reusable **File Upload Drop Zone** with drag & drop, progress, retry, cancel, and optimistic delete
- An **Attachment Gallery** for previewing, downloading, and batch actions
- A **Schedule Send Modal** with timezone-safe scheduling and smart quick picks

The implementation is fully typed, UI-consistent (Fluent UI), and designed to work with both **Outlook (Graph)** and **Gmail** backends.

---

## 1. FileUploadDropZone

### Purpose
Handles file selection and upload lifecycle, including:
- Drag & drop + click-to-upload
- Upload progress tracking
- Retry / cancel uploads
- Optimistic UI updates synced with parent state

### Key Props
| Prop | Type | Description |
|----|----|----|
| value | File[] | Authoritative uploaded files from parent |
| maxSize | number | Max file size (default 30MB) |
| accept | Accept | Allowed MIME types / extensions |
| multiple | boolean | Allow multiple uploads |
| onFilesDrop | fn | Upload handler with progress & cancel hooks |
| onDelete | fn | Delete handler (parent-driven) |
| onView | fn | View/preview handler |
| onDragStateChange | fn | Drag active state callback |

### Upload Lifecycle
1. User drops or selects files
2. Each file gets a **stable UID** (signature-based)
3. `onFilesDrop` is called with:
   - progress callback
   - cancel registration
4. UI reflects:
   - Uploading
   - Success
   - Failed (Retry enabled)
5. On delete:
   - Upload is cancelled if in-progress
   - File is removed optimistically
   - Parent `value` sync finalizes removal

### Design Highlights
- Signature-based de-duplication (`name + size`)
- Tombstone strategy to avoid flicker during parent updates
- Upload cancellation support
- Gradient progress bar with status-aware coloring

---

## 2. Attachment Gallery

### Purpose
Displays uploaded attachments in a responsive grid with:
- Image previews
- File-type icons
- Secure preview blocking for unsafe files
- Download and batch-download support

### Props
| Prop | Type | Description |
|----|----|----|
| attachments | any[] | Attachment metadata list |
| maxVisible | number | Initial visible count |
| onView | fn | Preview handler |
| onDownload | fn | Download handler |
| onDownloadAll | fn | Optional batch download |

### Security Rules
The gallery **blocks preview** for:
- Executables (`.exe`, `.dll`, etc.)
- Archives (`.zip`, `.rar`, `.7z`, etc.)

These files remain downloadable but not previewable.

### UX Behavior
- Hover overlay with actions (View / Download)
- Filename + size caption
- Expand / collapse attachment list
- Total size shown for large attachment sets

---

## 3. Schedule Send Modal

### Purpose
Allows users to schedule email delivery using **timezone-correct logic** with smart defaults.

### Features
- Timezone-safe scheduling using "fake UTC" strategy
- 5-minute granularity time slots
- Validation against past times
- Keyboard-accessible navigation

### Quick Picks
- This noon
- This evening
- Tomorrow morning
- Later this week / Monday morning

Quick picks automatically:
- Adjust for timezone
- Skip invalid times
- Submit immediately

### Data Contract
```ts
ScheduleValues {
  followUpDate: "DD/MM/YYYY"
  followUpHour: "01".."12"
  followUpMinute: "00".."55"
  followUpTT: "AM" | "PM"
}
```

### Validation Rules
- Date must be today or later
- Time must be in the future
- Invalid or exhausted dates are disabled
- Error feedback shown inline

---

## Accessibility & UX
- Keyboard navigation supported throughout
- ARIA labels on interactive elements
- Focus trapping inside modal
- Graceful fallback for unsupported previews

---

## Integration Notes
- Designed to plug into **Compose / Reply / Forward** flows
- Upload handler is backend-agnostic
- Works with Graph chunk upload & Gmail multipart upload
- Schedule values map directly to backend scheduling APIs

---

## Summary
This module delivers a **production-ready attachment and scheduling system** with:
- Strong UX
- Robust state management
- Security-aware previews
- Timezone-safe scheduling

It is reusable, extensible, and aligned with enterprise email workflows.

