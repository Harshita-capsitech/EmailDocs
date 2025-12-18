

# Team Report Documentation

## Overview
The **Team Report** module aggregates and presents time usage, efficiency, and availability for each team member. This report is designed for managers and administrators to monitor team performance over a specified period.

### Key Features
- Displays time usage statistics for each team member, including total time, time used, and available time.
- Calculates team efficiency based on fixed daily time versus actual time used.
- Allows for filtering by date range, including options to include or exclude weekends.

## DFD (Data Flow Diagram)
```mermaid
---
config:
  layout: dagre
---
flowchart TB
    AdminUser["Admin or Manager User"] e1@-.-> UI["Browser UI"]
    UI e2@--> Reports["Overview & Reports"]
    UI L_UI_Communication_0@-.-> Communication["Communication"] & Sales["Quotes & Sales"] & UserStats["User Stats"]
    Reports e6@-.-> Contact["Contact"] & BusinessInvoice["Business Invoice"]
    Communication e8@==> PhoneCall["Phone Call"]
    Sales e9@-.-> BusinessInvoice
    UserStats e10@-.-> Reports

    style AdminUser fill:#FFFFFF,stroke:#616161
    style UI stroke:#616161
    style Reports stroke:#757575
    style Communication stroke:#616161
    style Sales stroke:#757575
    style UserStats stroke:#616161
    style Contact stroke:#757575
    style BusinessInvoice stroke:#757575
    style PhoneCall stroke:#757575

    e1@{ animate: true } 
    e2@{ animate: true, curve: natural } 
    L_UI_Communication_0@{ curve: natural, animation: fast } 
    L_UI_Sales_0@{ animation: fast } 
    L_UI_UserStats_0@{ animation: fast } 
    e6@{ animate: true } 
    L_Reports_BusinessInvoice_0@{ animation: fast } 
    e8@{ animate: true } 
    e9@{ animate: true } 
    e10@{ animate: true, curve: natural }
```
## Process Flow

### 1) Admin/Manager Accesses Dashboard

```mermaid
---
config:
  flowchart:
    curve: monotoneY
  theme: neutral
  look: classic
---
flowchart LR
    A["Start"] e1@==> step1(["Admin/Manager 
Accesses Dashboard"])
    step1 e2@==> step2["Dashboard 
Requests Data (API call)"]
    step2 e3@==> step3["Backend Aggregation
from UserSessionPageView"]
    step3 e4@==> step4["Frontend 
Receives Aggregated Data"]
    step4 e5@==> step5["Display 
Team/Member Report"]
    step5 e6@==> B["End"]

    A@{ shape: sm-circ}
    step2@{ shape: fr-rect}
    step3@{ shape: cyl}
    step4@{ shape: rect}
    step5@{ shape: curv-trap}
    B@{ shape: dbl-circ}
    style step1 fill:#FFFFFF,stroke:#424242
    style step2 fill:#FFFFFF
    style step3 fill:#FFFFFF,stroke:#757575,stroke-width:2px
    style step4 fill:#FFFFFF
    style step5 fill:#FFFFFF,stroke:#757575,stroke-width:2px

    e1@{ animate: true } 
    e2@{ animate: true } 
    e3@{ animate: true } 
    e4@{ animate: true } 
    e5@{ animate: true } 
    e6@{ animate: true }
```

## ER Diagram
```mermaid
erDiagram
    PRACTICE {
        int practiceId PK "Root Entity"
    }
    APPLICATION_USER {
        string _id PK
        string Name
        string[] Roles
    }
    USER_TEAM {
        string _id PK
        string TeamName
    }
    COMPANY {
        string _id PK
        string CompanyName
    }
    CONTACT {
        string _id PK
        int Type "Lead/Customer"
    }
    BUSINESS_INVOICE {
        string _id PK
        Amount Amount
        int Status
    }
    PHONE_CALL {
        string _id PK
        decimal Duration
    }

    PRACTICE ||--o{ USER_TEAM : "has team"
    USER_TEAM ||--o{ APPLICATION_USER : "includes"
    APPLICATION_USER ||--o{ CONTACT : "manages"
    COMPANY ||--o{ CONTACT : "has contact"
    COMPANY ||--o{ BUSINESS_INVOICE : "billed to"
    APPLICATION_USER ||--o{ PHONE_CALL : "conducts"
```
## Entity Definitions

- **TeamMemberReport**: Represents the individual report for each team member.
  - `Member`: The team member's name and ID.
  - `TotalTime`: The total fixed time allocated for the member (8 hours 30 minutes per day).
  - `TimeUsed`: The actual time the member worked.
  - `TimeAvailable`: The remaining time the member could potentially work.
  - `TeamEfficiency`: The efficiency percentage based on actual time worked versus allocated time.

- **TeamMemberReportSummary**: Represents the overall summary of the team.
  - `TotalTeams`: The total number of teams.
  - `TotalHours`: The total available hours for all team members combined.
  - `AverageHours`: The average hours worked per team member.
  - `TotalTimeAvailable`: The total available time for the team.
  - `TeamEfficiency`: The overall efficiency percentage for the team.

## Authentication / APIs

### Authentication
The **Team Report** endpoint requires an **ADMIN** or **MANAGER** role to access. The backend is protected using role-based access control (RBAC) with the `[Authorize]` attribute.

### API Endpoints

- **GetTeamReport (Backend)**  
  **Method:** `GET`  
  **Endpoint:** `/teamsreport/{teamId}/TeamReport`  
  **Query Parameters:**
  - `teamId`: The ID of the team to retrieve the report for.
  - `fromDate`: The start date for the report.
  - `toDate`: The end date for the report.
  - `isWeekendIncluded`: A boolean flag to include/exclude weekends.

### Frontend API Call:
- **Route**: `/teamsreport`
- **Method**: `GET`
- **Parameters**:
  - `start`, `length`, `search`, `sortCol`, `sortDir`, `fromDate`, `toDate`

### Backend Method:
- **Route**: `/{teamId}/TeamReport`
- **Method**: `GET`
- **Parameters**:
  - `teamId`: The ID of the team
  - `fromDate`: Start date for the report (optional)
  - `toDate`: End date for the report (optional)
  - `isWeekendIncluded`: Boolean flag to include/exclude weekends (optional)


## Testing Guide

1. **Unit Tests**:  
   The backend API can be unit tested by mocking database calls to simulate various scenarios, such as valid/invalid team IDs and date ranges. Ensure that the time calculations are correct, especially for team efficiency.

2. **Frontend Tests**:  
   Test the frontend using component testing tools like **React Testing Library**. Ensure that the time filters work correctly and that the team data is fetched and displayed properly.

3. **Integration Tests**:  
   Ensure that the frontend can communicate with the backend API and that the correct data is displayed on the dashboard.

## References
- **API Documentation**:  
   Refer to the backend API documentation for detailed information about the Team Report endpoints and data structures.
  
- **Frontend Code**:  
   The frontend code for fetching and displaying the team report is located in `NewTeamsReport.tsx`. The frontend interacts with the backend API to retrieve and display data based on the selected filters.

---

