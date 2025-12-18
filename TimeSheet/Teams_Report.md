

# Team Report Documentation

## Overview
The **Team Report** module aggregates and presents time usage, efficiency, and availability for each team member. This report is designed for managers and administrators to monitor team performance over a specified period.

### Key Features
- Displays time usage statistics for each team member, including total time, time used, and available time.
- Calculates team efficiency based on fixed daily time versus actual time used.
- Allows for filtering by date range, including options to include or exclude weekends.

## DFD (Data Flow Diagram)
```mermaid
flowchart TD
  AdminUser["Admin or Manager User"]
  UI["Browser UI"]
  Reports["Overview & Reports"]
  Communication["Communication"]
  Sales["Quotes & Sales"]
  UserStats["User Stats"]
  
  Contact["Contact"]
  PhoneCall["Phone Call"]
  BusinessInvoice["Business Invoice"]

  AdminUser e1@==> UI
  UI e2@==> Reports
  UI e3@==> Communication
  UI e4@==> Sales
  UI e5@==> UserStats
  Reports e6@==> Contact
  Reports e7@==> BusinessInvoice
  Communication e8@==> PhoneCall
  Sales e9@==> BusinessInvoice
  UserStats e10@==> Reports

  e1@{ animate: true }
  e2@{ animate: true }
  e3@{ animate: true }
  e4@{ animate: true }
  e5@{ animate: true }
  e6@{ animate: true }
  e7@{ animate: true }
  e8@{ animate: true }
  e9@{ animate: true }
  e10@{ animate: true }
```
## Process Flow

### 1) Admin/Manager Accesses Dashboard

```mermaid
%%{init: {"flowchart":{"curve":"monotoneY"}} }%%
flowchart TB
    A@{ shape: sm-circ, label: "Start" }

    step1@{ shape: stadium, label: "Admin/Manager \nAccesses Dashboard" }
    step2@{ shape: fr-rect, label: "Dashboard \nRequests Data (API call)" }
    step3@{ shape: cyl, label: "Backend Aggregation\nfrom UserSessionPageView" }
    step4@{ shape: rect, label: "Frontend \nReceives Aggregated Data" }
    step5@{ shape: curv-trap, label: "Display \nTeam/Member Report" }
    B@{ shape: dbl-circ, label: "End" }

    A e1@==> step1
    step1 e2@==> step2
    step2 e3@==> step3
    step3 e4@==> step4
    step4 e5@==> step5
    step5 e6@==> B

    e1@{ animate: true }
    e2@{ animate: true }
    e3@{ animate: true }
    e4@{ animate: true }
    e5@{ animate: true }
    e6@{ animate: true }

    %% Optional: add style for clarity
    style step3 fill:#d9f6fb,stroke:#09999a,stroke-width:2px
    style step5 fill:#faeec7,stroke:#e1aa13,stroke-width:2px
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

