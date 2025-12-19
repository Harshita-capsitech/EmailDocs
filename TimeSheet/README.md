# TimeSheet System Documentation

## Overview
Comprehensive documentation for the ActingOffice TimeSheet System covering all modules, reports, and analytical features for tracking employee productivity, time utilization, and performance metrics.

## Documentation Structure

### Module Documentation

| Module | Description | Documentation |
|--------|-------------|---------------|
| **[Dashboard](./Dashboard.md)** | Central hub for monitoring key performance metrics | Overview, user stats, communication, sales |
| **[User Report](./User_Report.md)** | Individual user timesheet analysis | Module-wise time tracking, detailed activity logs |
| **[Teams Report](./Teams_Report.md)** | Team performance and efficiency metrics | Team efficiency, member performance, availability |
| **[Modules Report](./Modules_Report.md)** | Activity tracking across system modules | Tax, Payroll, Accounts, CRM, and other modules |
| **[Clients Report](./Clients_Report.md)** | Client-wise time allocation and engagement | Time spent per client, team assignments |
| **[Calls & Emails Report](./Calls_Emails.md)** | Communication activity tracking | Call logs, email statistics, task completion |

## Quick Navigation

### By Business Function

**Performance Tracking**:
1. [View Dashboard Overview](./Dashboard.md)
2. [Analyze User Performance](./User_Report.md)
3. [Review Team Efficiency](./Teams_Report.md)
4. [Track Module Activity](./Modules_Report.md)

**Client Management**:
1. [Client Time Analysis](./Clients_Report.md)
2. [Communication Tracking](./Calls_Emails.md)
3. [Engagement Metrics](./Dashboard.md#sales-and-quotes)

**Administrative Functions**:
- [User Statistics](./Dashboard.md#user-stats)
- [Team Management](./Teams_Report.md)
- [Report Configuration](./Dashboard.md#filters)

### By User Role

#### Admin/Manager Access
- **Dashboard**: Complete overview of organizational metrics
- **User Reports**: Detailed individual performance analysis
- **Teams Reports**: Team efficiency and productivity tracking
- **Modules Reports**: Cross-module activity analysis
- **Clients Reports**: Client engagement and time allocation
- **Calls & Emails**: Communication statistics and task tracking

#### Staff Access
- **Personal Timesheet**: Individual time tracking
- **Module Activity**: Personal module usage statistics
- **Task Completion**: Personal task metrics

## Key Features

### Real-Time Analytics
- **Live Dashboard**: Real-time updates of key metrics
- **Time Tracking**: Accurate activity and idle time monitoring
- **Module Analytics**: Detailed breakdown by system modules
- **Team Insights**: Performance comparison across teams

### Comprehensive Reporting
- **User Performance**: Individual productivity metrics
- **Team Efficiency**: Calculated as (Time Used / Time Available) × 100
- **Module Distribution**: Time allocation across different modules
- **Client Engagement**: Time spent per client with drill-down capability
- **Communication Stats**: Incoming/outgoing calls and email metrics

### Filtering & Customization
- **Date Range Selection**: Custom periods for analysis
- **Team Filtering**: Focus on specific teams
- **User Selection**: Individual or group analysis
- **Weekend Options**: Include/exclude weekends
- **Sorting Options**: Multiple column sorting

### Drill-Down Capabilities
- **Detailed Views**: Click-through to granular data
- **Page Views**: Session-level activity tracking
- **Call Details**: Individual call records
- **Email Details**: Email-by-email breakdown
- **Task Details**: Completed task listings

## System Architecture

### Technology Stack
- **Frontend**: React with TypeScript
- **Backend**: .NET Core API
- **Database**: MongoDB with aggregation pipelines
- **Authentication**: Role-Based Access Control (RBAC)
- **API Documentation**: Swagger/OpenAPI

### Data Flow
```
User → React UI → API Controller → Backend Service → MongoDB → Aggregation → Response
```

### Key Collections
- **ApplicationUserDB**: User profiles and authentication
- **UserSessionPageView**: Activity tracking data
- **PhoneCallDB**: Call records and statistics
- **EmailDB**: Email communication logs
- **TeamDB**: Team structure and assignments
- **TaskDB**: Task completion records
- **Companies**: Client information

## Authentication & Authorization

All timesheet endpoints require authentication with appropriate roles:

| Role | Access Level |
|------|--------------|
| **ADMIN** | Full access to all reports and data |
| **MANAGER** | Team-specific and organizational reports |
| **STAFF** | Personal timesheets and statistics |

## API Routes

All timesheet APIs follow consistent routing:

```
/api/timesheet/[endpoint]
```

### Core Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/UserStats` | GET | Dashboard user statistics |
| `/OverviewReport` | GET | Consolidated overview metrics |
| `/Communication` | GET | Phone calls and communication data |
| `/QuotesSales` | GET | Sales and quotes statistics |
| `/userreport` | GET | Individual user detailed report |
| `/TeamsReport` | GET | Teams list for reporting |
| `/teamsreport/{teamId}/TeamReport` | GET | Specific team performance |
| `/ModuleReport` | GET | Module-wise activity report |
| `/clientsreport` | GET | Client engagement metrics |
| `/Report` | GET | Calls & emails comprehensive report |

## Module Breakdown

### 1. Dashboard Module (4 Key Areas)
- **Overview Report**: Leads, calls, emails, website queries
- **User Statistics**: Top/bottom performers, time tracking
- **Communication Overview**: Incoming/outgoing call statistics
- **Sales & Quotes**: Business performance metrics

### 2. User Report Module (3 Components)
- **Module Activity**: Time spent across all modules
- **Session Tracking**: Detailed page view records
- **Performance Metrics**: Total time, efficiency calculations

### 3. Teams Report Module (4 Metrics)
- **Team Efficiency**: Overall team performance percentage
- **Member Statistics**: Individual contribution to team goals
- **Time Allocation**: Available vs. used time analysis
- **Comparative Analysis**: Team-to-team performance comparison

### 4. Modules Report Module (8+ Modules)
- Tax & Accounts
- Accounts
- CT600
- SA100
- Payroll
- Bookkeeping
- CRM
- Others

### 5. Clients Report Module (3 Views)
- **Client List**: All clients with time metrics
- **Time Allocation**: Total time spent per client
- **Drill-Down**: Detailed activity per client

### 6. Calls & Emails Module (5 Metrics)
- **Call Statistics**: Incoming/outgoing/total calls
- **Email Statistics**: Sent/received/draft emails
- **Task Completion**: Tasks completed by user
- **Average Metrics**: Average call/email duration
- **Time Totals**: Total communication time

## Common Workflows

### Daily Activity Review
```
1. Open Dashboard
2. Review User Stats table
3. Check communication overview
4. Monitor sales/quotes updates
5. Identify performance outliers
```

### Weekly Team Analysis
```
1. Navigate to Teams Report
2. Select date range (last 7 days)
3. Review team efficiency metrics
4. Compare member performance
5. Identify improvement areas
```

### Monthly Performance Review
```
1. Generate User Report for each team member
2. Analyze module-wise time distribution
3. Review client engagement metrics
4. Evaluate communication efficiency
5. Compile findings for review meeting
```

### Client Time Analysis
```
1. Open Clients Report
2. Filter by client or team
3. Review time spent metrics
4. Drill down to detailed activities
5. Generate billing reports
```

## Best Practices

✅ **Regular Monitoring**: Check dashboard daily for real-time insights  
✅ **Team Reviews**: Weekly team report analysis  
✅ **User Feedback**: Monthly performance reviews with staff  
✅ **Data Accuracy**: Ensure timely activity logging  
✅ **Filter Usage**: Leverage filters for focused analysis  
✅ **Drill-Down**: Use detail views for investigation  
✅ **Trend Analysis**: Compare periods for performance trends  
✅ **Action Plans**: Convert insights into improvement actions  

## Performance Metrics

### Key Performance Indicators (KPIs)

**Individual Metrics**:
- Total Active Time
- Module Distribution
- Tasks Completed
- Communication Volume
- Client Engagement

**Team Metrics**:
- Team Efficiency Percentage
- Average Member Performance
- Time Availability
- Workload Distribution

**Business Metrics**:
- Lead Generation
- Conversion Rates
- Sales Performance
- Client Satisfaction (via response time)

## Report Scheduling

### Recommended Frequency

| Report | Frequency | Purpose |
|--------|-----------|---------|
| Dashboard | Daily | Quick performance overview |
| User Report | Weekly | Individual performance tracking |
| Teams Report | Weekly | Team efficiency monitoring |
| Modules Report | Monthly | Resource allocation analysis |
| Clients Report | Monthly | Client engagement review |
| Calls & Emails | Bi-weekly | Communication effectiveness |

## Data Retention

- **Activity Logs**: 12 months minimum
- **Aggregated Reports**: 24 months
- **User Statistics**: Indefinite (historical trending)
- **Call Records**: 12 months
- **Email Logs**: 12 months

## Troubleshooting

### Common Issues

**Issue**: Report shows no data
- **Solution**: Check date range, team filter, user permissions

**Issue**: Efficiency percentage seems incorrect
- **Solution**: Verify time available calculation, check for excluded weekends

**Issue**: Module times don't add up to total
- **Solution**: Check for overlapping sessions, idle time exclusions

**Issue**: User not appearing in report
- **Solution**: Verify user has activity in selected period, check team assignment

## Support & Resources

- **Technical Documentation**: This documentation index
- **API Reference**: [Swagger UI](https://apiuat.actingoffice.com/api-docs/index.html?urls.primaryName=Acting+Office+-+CRM)
- **User Guides**: Individual report documentation files
- **Training Materials**: Contact system administrator

## Getting Started

### For Administrators
1. Review [Dashboard Overview](./Dashboard.md) to understand key metrics
2. Study [Teams Report](./Teams_Report.md) for team management
3. Configure user permissions and team assignments
4. Set up preferred filters and date ranges
5. Schedule regular report reviews

### For Managers
1. Familiarize with [Teams Report](./Teams_Report.md) for team monitoring
2. Learn [User Report](./User_Report.md) for individual analysis
3. Review [Modules Report](./Modules_Report.md) for workload distribution
4. Use [Clients Report](./Clients_Report.md) for client engagement
5. Monitor [Calls & Emails Report](./Calls_Emails.md) for communication

### For Developers
1. Review API documentation for each module
2. Understand MongoDB aggregation pipelines
3. Study authentication and authorization flows
4. Test endpoints using provided test cases
5. Review ER diagrams for data relationships

## Testing Guidelines

Each module includes comprehensive testing documentation:

### Unit Testing
- Component rendering tests
- Data calculation validation
- Filter and sorting logic
- Time conversion functions

### Integration Testing
- API endpoint responses
- Database query accuracy
- Filter application correctness
- Pagination functionality

### End-to-End Testing
- Complete user workflows
- Data flow from UI to database
- Report generation accuracy
- Export functionality

## Version Information

**Documentation Version**: 1.0  
**Last Updated**: December 2025  
**System Version**: ActingOffice TimeSheet v1.0  
**Maintained By**: ActingOffice Development Team

## Documentation Standards

All report documentation includes:
1. **Overview** - Purpose and key features
2. **Data Flow Diagram (DFD)** - Visual data flow representation
3. **Process Flow Diagram** - Step-by-step workflow
4. **ER Diagram** - Entity relationships
5. **Entity Definitions** - Data structure details
6. **Authentication/APIs** - Endpoints and authorization
7. **Testing Guide** - Comprehensive test scenarios

## Contributing

To update documentation:
1. Follow existing structure and formatting
2. Include all required sections
3. Update version information
4. Add mermaid diagrams where applicable
5. Include API endpoint changes
6. Update testing guides

## License

Copyright © Capsitech/ActingOffice. All rights reserved.
