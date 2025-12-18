

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

    AdminUser --> UI
    UI --> Reports
    UI --> Communication
    UI --> Sales
    UI --> UserStats
    Reports --> Contact
    Reports --> BusinessInvoice
    Communication --> PhoneCall
    Sales --> BusinessInvoice
    UserStats --> Reports
```
## Process Flow

1. **Admin/Manager Accesses Dashboard:**
   - The manager accesses the team report dashboard to view the time usage and efficiency data for the selected team(s).
   
2. **Request Data:**
   - The dashboard requests data from the backend APIs for the selected time range.
   
3. **Data Retrieval:**
   - The backend processes the request, aggregating data from the `UserSessionPageView` collection, filtering by team and date range, and calculating time usage and efficiency.
   
4. **Data Response:**
   - The frontend receives the aggregated data, including team member time usage, total available time, and efficiency metrics.
   
5. **Display Report:**
   - The data is displayed in the dashboard, where the admin can review the report for individual members and the overall team.

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
   The frontend code for fetching and displaying the team report is located in `NewTeamsReport.tsx`. The frontend interacts with the backend API to retrieve and display data based on the selected filters 【10†source】.

---

End of Documentation
