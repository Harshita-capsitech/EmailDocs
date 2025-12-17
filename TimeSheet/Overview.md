#  Timesheet Dashboard API & Frontend Module Documentation

##  Overview

The **Timesheet Dashboard** is an Admin/Manager reporting experience that aggregates **user activity time**, **communications**, **leads**, **website queries**, and **quotes/sales** into a single dashboard UI.

It consists of:
- **Backend reporting endpoints** (MongoDB aggregations) that compute overview KPIs and breakdowns.
- **Frontend dashboard components** that render cards, charts, and the “Top 10 / Bottom 10” user stats table.

On the UI, the dashboard container (`NewAdminTimeSheet`) drives the overall date/team filters and refresh behavior, then calls the reporting APIs and passes data to the child cards and tables. 

---

## DFD

### Level 0 Data Flow Diagram

```mermaid
---
config:
  theme: mc
---
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
    BusinessQuote["Business Quote"]
    ApplicationUser["Application User"]
    Companies["Companies"]

    AdminUser admin2ui@--> UI
    UI ui2rep@--> Reports
    UI ui2comm@--> Communication
    UI ui2sales@--> Sales
    UI ui2stats@--> UserStats

    Reports rep2con@--> Contact
    Communication comm2call@--> PhoneCall
    Sales sales2inv@--> BusinessInvoice
    Sales sales2quo@--> BusinessQuote
    UserStats stat2user@--> ApplicationUser
    UserStats stat2com@--> Companies

    %% Animate the key edges
    admin2ui@{ animation: fast }
    ui2rep@{ animation: slow }
    ui2comm@{ animation: slow }
    ui2sales@{ animation: slow }
    ui2stats@{ animation: slow }
    rep2con@{ animate: true }
    comm2call@{ animate: true }
    sales2inv@{ animate: true }
    sales2quo@{ animate: true }
    stat2user@{ animate: true }
    stat2com@{ animate: true }
```

---

## Process Flow

### 1) Admin Timesheet Page Load

```mermaid
flowchart LR
    %% Neutral class definitions
    classDef ui fill:#e3ecfa,stroke:#657693,stroke-width:2px;
    classDef service fill:#e5f7ef,stroke:#3f908d,stroke-width:2px;
    classDef api fill:#e7ebed,stroke:#425057,stroke-width:2px;

    %% Nodes
    UI(NewAdminTimeSheet UI):::ui
    API(Backend API):::api

    subgraph Middleware ["Middleware Services Layer"]
        direction TB
        RS(ReportService):::service
        PCS(PhoneCallService):::service
        TS(TimesheetService):::service
    end

    %% Animated Connections - UI to Middleware
    UI ui2rs@-->|Overview, Sales & Quotes| RS
    UI ui2pcs@-->|Comm Stats| PCS
    UI ui2ts@-->|User Stats| TS

    %% Animated Connections - Middleware to API
    RS rs2api@-->|/Timesheet/Overview & Quotes| API
    PCS pcs2api@-->|/Timesheet/Communication| API
    TS ts2api@-->|/Timesheet/UserStats| API

    %% -- Animate the edges
    ui2rs@{ animation: slow }
    ui2pcs@{ animation: slow }
    ui2ts@{ animation: slow }
    rs2api@{ animation: slow }
    pcs2api@{ animation: slow }
    ts2api@{ animation: slow }
```

### 2) UserStats (Top/Bottom Users) — High-level Logic

**Goal:** Show only users who have meaningful activity, then compute relative position vs average.

1. Parse date range.
2. Load candidate users for the practice (optionally team-filtered).
3. Aggregate activity totals per user (sum of module activity totals) for the date range.
4. Enrich user list with their `TotalTime`.
5. Remove users with `< 60 seconds` activity.
6. Sort by `TotalTime` (asc/desc depending on “Top” vs “Bottom”).
7. Compute average percent indicator per user (relative to overall average).
8. Take 10% of the list (current code: `users.Count * 10 / 100`) and assign serial numbers.

---

##  ER Diagram

> **Movable/zoomable note:** Mermaid diagrams can be panned/zoomed in many Markdown viewers (GitHub, Mermaid Live, VSCode extensions). If your viewer is static, open the `.md` in a Mermaid-enabled renderer for drag/zoom.

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

---

## Entity Definition

### Response / DTO models used by the dashboard

#### `TimeSheetReport`
Aggregated overview KPIs for the top cards.
- `LeadReport` (`LeadReportResponse`): leads, converted, lost
- `CallReport` (`CallReportResponse`): calls, incoming, outgoing, duration
- `WebQueries` (`WebQueriesResponse`): total and by query type
- `TimesheetMails` (`TimesheetMailResponse`): email counts (currently returned as zeros in the shown implementation)

#### `CommunicationOverviewResponse`
- `CallsOverview` (`ComunicationCallResponse`): totals + time-bucketed incoming/outgoing series
- `ReviewsOverview` (`ReviewsOverview`): placeholder object currently initialized with zeros in the shown backend code

#### `TSDashBoardUserStats`
User-level “Top/Bottom” list item
- `Name` (concatenated first + last)
- `Teams` (`ApplicationUserTeamDetail[]`)
- `TotalTime` (seconds)
- `Average` (computed “above average” indicator value)

#### `TeamMemberReport` / `TeamMemberReportSummary`
Team rollup report:
- `TotalTime` = fixed capacity (8h 30m per working day × number of working days)
- `TimeUsed` = aggregated active seconds
- `TimeAvailable` = max(0, capacity - used)
- `TeamEfficiency` = used% capped at 100

---

## Authentication / APIs

### Authentication
All endpoints are protected by your API’s authentication middleware. Some endpoints have explicit role requirements:
- `UserReport` requires **ADMIN** role.
- `TeamReport` requires **ADMIN** or **MANAGER**.

Use a JWT token:

```http
Authorization: Bearer <jwt_token>
Content-Type: application/json
```

### Base URL
```txt
https://<your-domain>/<controller-base>
```

### Endpoints

#### 1) Dashboard User Stats (Top/Bottom list)
```http
GET /Timesheet/UserStats?fromDate=DD/MM/YYYY&toDate=DD/MM/YYYY&teamId=<id>&sortDir=asc|desc
```

**Notes**
- Filters users by practice, excludes certain statuses/roles, excludes `Config.EFR == true`.
- Aggregates `UserDashboardData.Date` between from/to and sums `ModuleActivity.Total`.
- Removes users with `TotalTime < 60` seconds.
- Computes `Average` using `GetAvereage(totalTime, averageTime)`.

#### 2) PageViews User Report (user-wise client report)
```http
GET /Timesheet/UserReport?userId=<id>&start=0&length=15&sortCol=Name&sortDir=asc&fromDate=dd/MM/yyyy&toDate=dd/MM/yyyy&isWeekendIncluded=false
```

**Notes**
- Aggregates `UserSessionPageView` per `CompanyId`
- If weekends excluded, adds a `$dayOfWeek` field and filters out 1 (Sunday) and 7 (Saturday)
- Looks up `Companies` to attach `Name` and `CompanyNumber`

#### 3) Communication Overview (Calls chart + Reviews summary)
```http
GET /Timesheet/Communication?fromDate=dd/MM/yyyy&toDate=dd/MM/yyyy&teamId=<optional>
```

**Notes**
- Groups calls by `Direction` for totals.
- Chooses a dynamic “bucket strategy” based on date span: weekday vs date vs month.
- Returns an ordered bucket list with missing buckets filled (`EnsureContinuous`).
- Frontend renders it as a bar chart. fileciteturn0file1L148-L173

#### 4) Overview Report (Leads, Calls, Web Queries, Emails)
```http
GET /Timesheet/OverviewReport?teamId=<optional>&fromDate=dd/MM/yyyy&toDate=dd/MM/yyyy
```

**Notes**
- Runs three aggregates in parallel: Contacts, PhoneCalls, WebsiteQueries.
- Builds `TimeSheetReport` from grouped results.
- Email stats currently returned as zeros in the shown backend snippet.

#### 5) Quotes & Sales Report
```http
GET /Timesheet/QuotesSales?fromDate=dd/MM/yyyy&toDate=dd/MM/yyyy&teamId=<optional>
```

**Notes**
- Sales: filters invoices (non-void) and builds time-series buckets for receipts vs pending
- Quotes: groups quotes by status to compute `sentQuotes`, `acceptedQuotes`, `pendingQuotes`
- Frontend renders Sales as stacked bars and Quotes as donut chart. fileciteturn0file2L214-L276

#### 6) Team Report
```http
GET /Timesheet/{teamId}/TeamReport?fromDate=dd/MM/yyyy&toDate=dd/MM/yyyy&isWeekendIncluded=false
```

**Notes**
- Pulls team users/leaders from `UserTeam`
- Aggregates `UserSessionPageView.TotalActive` by user
- Computes capacity using 8h30m/day × working days

---

## Testing Guide

### Quick smoke tests (curl)

> Replace `BASE_URL` and token, and use the date format your API expects.

```bash
export BASE_URL="https://apidev.example.com/Timesheet"
export TOKEN="YOUR_JWT"

curl -s "$BASE_URL/OverviewReport?fromDate=01/12/2025&toDate=31/12/2025&teamId=" \
  -H "Authorization: Bearer $TOKEN" | jq

curl -s "$BASE_URL/QuotesSales?fromDate=01/12/2025&toDate=31/12/2025&teamId=" \
  -H "Authorization: Bearer $TOKEN" | jq

curl -s "$BASE_URL/Communication?fromDate=01/12/2025&toDate=31/12/2025&teamId=" \
  -H "Authorization: Bearer $TOKEN" | jq

curl -s "$BASE_URL/UserStats?fromDate=01/12/2025&toDate=31/12/2025&teamId=&sortDir=desc" \
  -H "Authorization: Bearer $TOKEN" | jq
```

### Functional test checklist
- Date range filter returns stable bucket sequences (no missing labels).
- Team filter scopes results consistently across all cards.
- Switching date/team should not cause redundant double-fetches on the same component (watch network tab).
- Validate weekend exclusion behavior in `UserReport` and `TeamReport`.
- Verify role-gated endpoints return **403** for non-admin/non-manager accounts.

---

##  References

### Frontend modules (uploaded)
- Dashboard container: `NewAdminTimeSheet.tsx` fileciteturn0file0L1-L23  
- Communication + Reviews cards: `NewCommunicationReviewsCardComponent.tsx` fileciteturn0file1L1-L23  
- Sales + Quotes cards: `NewSalesQuotesCardComponent.tsx` fileciteturn0file2L1-L20  
- Top/Bottom user table: `NewTimeSheetUserTable.tsx` fileciteturn0file3L1-L23  

### Notes
- The dashboard triggers API calls via `useEffect` based on dependencies like `fromDate`, `toDate`, `refresh`, and `teamId`. fileciteturn0file1L147-L186  
- Sales/Quotes depend on `ReportService.getQuotesAndSalesReport(...)` and render ECharts charts. fileciteturn0file2L175-L213  
