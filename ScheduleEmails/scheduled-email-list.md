# Scheduled Email List Documentation

## Overview
The **Scheduled Email List** feature provides an interface for users to manage their scheduled emails. It allows users to view a list of all scheduled emails, filter by various parameters (e.g., status, user, company), and perform actions such as rescheduling or moving emails to drafts. This feature is part of the broader email scheduling system, offering powerful tools for organizing and managing email communications.

---

## DFD (Data Flow Diagram)
```mermaid
flowchart TD
%% External Entities
User[(User)]
EmailService[[Send Email Service]]
ScheduleDB[(Scheduled Email Database)]
FailedScheduleDB[(Failed Scheduled Emails Database)]
LogService[(Logging Service)]

%% Processes
P1((Authenticate User))
P2((Fetch Scheduled Emails))
P3((View Email Details))
P4((Reschedule Email))
P5((Move Email to Draft))
P6((View Logs))

%% Data Flows
User animate1@--> P1
P1 animate2@--> EmailService animate3@--> P1
P1 animate4@--> P2
P2 animate5@--> ScheduleDB animate6@--> P2
P2 animate7@--> P3
P3 animate8@--> ScheduleDB
P3 animate9@--> FailedScheduleDB
FailedScheduleDB animate10@--> P5
P5 animate11@--> EmailService
P3 animate12@--> LogService
LogService animate13@--> P6

%% Animation Styles
animate1@{ animate: fast }
animate2@{ animate: fast }
animate3@{ animate: fast }
animate4@{ animate: fast }
animate5@{ animate: fast }
animate6@{ animate: fast }
animate7@{ animate: fast }
animate8@{ animate: fast }
animate9@{ animate: fast }
animate10@{ animate: fast }
animate11@{ animate: fast }
animate12@{ animate: fast }
animate13@{ animate: fast }

%% Styling
style User stroke-width: 2px
style EmailService stroke-width: 2px
style ScheduleDB stroke-width: 2px
style FailedScheduleDB stroke-width: 2px
style LogService stroke-width: 2px

