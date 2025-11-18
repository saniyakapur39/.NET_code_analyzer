# .NET Application Functional & Integration Documentation

## 1. Functional Flow Diagrams

### 1.1 Pricing Approval Workflow
```mermaid
flowchart TD
    Start([User logs in])
    CheckRole{Is PRICING_SUPER_USER?}
    Dashboard[Show Pricing Approval Dashboard]
    Filter[User applies filters (status, request type, buying area, effective date)]
    Requests[Display filtered pricing requests]
    Select[User selects requests for approval]
    Approve[User clicks 'Approve Selected']
    BulkApprove[Bulk approval logic iterates requests]
    UpdateStatus[ProcBuilder.SubmitPricingApproval called for each request]
    Notify[EmailRevised.SendNotification triggers email]
    End([Redirect to Dashboard])
    Start --> CheckRole
    CheckRole -- Yes --> Dashboard
    CheckRole -- No --> Redirect[Redirect to Default.aspx]
    Dashboard --> Filter
    Filter --> Requests
    Requests --> Select
    Select --> Approve
    Approve --> BulkApprove
    BulkApprove --> UpdateStatus
    UpdateStatus --> Notify
    Notify --> End
```

### 1.2 Order Lifecycle (Exception/Original Price Request)
```mermaid
flowchart TD
    User[Buyer initiates price request]
    Validate[ProcBuilder validates request]
    Submit[Buyer submits request]
    DMMApproval[DMM reviews/approves/declines]
    PricingApproval[Pricing team reviews/approves/declines]
    NotifyBuyer[Email notifications sent to Buyer]
    UpdateStatus[Status updated in DB]
    End([Request lifecycle complete])
    User --> Validate
    Validate --> Submit
    Submit --> DMMApproval
    DMMApproval --> PricingApproval
    PricingApproval --> NotifyBuyer
    NotifyBuyer --> UpdateStatus
    UpdateStatus --> End
```

### 1.3 Bulk Operations (Batch Approvals)
```mermaid
flowchart TD
    Start([Approver logs in])
    SelectRequests[Select multiple requests]
    ApproveAll[Click 'Approve All']
    Iterate[Iterate requests]
    SubmitApproval[Submit approval for each]
    Notify[Send notification for each]
    End([Redirect to Dashboard])
    Start --> SelectRequests --> ApproveAll --> Iterate --> SubmitApproval --> Notify --> End
```

### 1.4 Dashboard & Reporting
```mermaid
flowchart TD
    User[User logs in]
    GetDetails[ProcBuilder.GetDashboardDetails]
    Filter[Apply filters]
    Display[Show dashboard grid]
    Export[User exports data]
    End([Dashboard interaction complete])
    User --> GetDetails --> Filter --> Display --> Export --> End
```

### 1.5 User Management & Role-Based Access
```mermaid
flowchart TD
    Login[User logs in]
    GetSession[MySession loads user details]
    CheckRole{UserRole/IsAdmin}
    GrantAccess[Grant page access]
    DenyAccess[Redirect to Default.aspx]
    Login --> GetSession --> CheckRole
    CheckRole -- Authorized --> GrantAccess
    CheckRole -- Unauthorized --> DenyAccess
```

### 1.6 Alerting & Notifications
```mermaid
flowchart TD
    Action[Business event triggers (approval, decline, etc.)]
    BuildEmail[Email.cs builds message]
    GetRecipients[Email.cs gets recipient list]
    SendSMTP[Email sent via SMTP]
    Log[Log notification event]
    Action --> BuildEmail --> GetRecipients --> SendSMTP --> Log
```

### 1.7 Exception Handling & Audit Logging
```mermaid
flowchart TD
    Event[Application/Workflow event]
    TryCatch[Try/Catch in DAL/Business Logic]
    LogError[ProcBuilder.WriteToLog or PCRDAL error handling]
    NotifyUser[Error surfaced to user]
    Audit[Audit entry created in DB]
    Event --> TryCatch --> LogError --> NotifyUser --> Audit
```

---

## 2. Core Business Functionalities

| Functionality Name         | Description                                                                 | Main Classes/Files                         | Key Business Rules/Validations                                         | Actors                    |
|---------------------------|-----------------------------------------------------------------------------|--------------------------------------------|-----------------------------------------------------------------------|---------------------------|
| Pricing Approval Workflow  | Approvers review, filter, and bulk approve pricing requests                 | PricingApproval.aspx.cs, ProcBuilder.cs    | Only PRICING_SUPER_USER can approve; status transitions; notifications | Pricing Approver          |
| Order Lifecycle           | Buyer submits price change request; DMM and Pricing approve/decline         | ProcBuilder.cs, Email.cs, Review.aspx.cs   | Validation of request; multi-step approval; email notifications        | Buyer, DMM, Pricing Team  |
| Bulk Operations           | Approvers can approve multiple requests in one action                       | PricingApproval.aspx.cs, ProcBuilder.cs    | Bulk approval iterates requests; notifications for each                | Pricing Approver          |
| Dashboard & Reporting     | Users view, filter, and export request data                                 | PricingDashboard.aspx.cs, ProcBuilder.cs   | Data filtered by role, status, date; export to Excel                   | All Users                 |
| User Management           | Session/user context, role-based access control                             | MySession.cs, ProcBuilder.cs               | Access checks on page load; admin roles; session initialization        | All Users, Admin          |
| Alerting & Notifications  | Email alerts for approvals, declines, deadlines                             | Email.cs, EmailRevised.cs, ProcBuilder.cs  | Dynamic recipient lists; message content based on workflow status      | All Users                 |
| Inventory Management      | Style/SKU reconciliation, uploads, validations                              | ProcBuilder.cs, Upload.cs                  | Validation of SKUs/styles; error handling for duplicates, invalid data | Buyer, Approver           |
| Supplier Onboarding       | Adding visibility users, chain management                                   | ProcBuilder.cs                             | Add/delete visibility users; chain updates                             | Admin, Buyer              |
| Exception Handling & Audit| Error catching, logging, audit trail                                        | ProcBuilder.cs, PCRDAL.cs, Global.asax.cs  | Log errors, audit user actions, surface errors to UI                   | All Users, Admin          |
| Administrative Utilities  | Maintenance, chain updates, effective date management                       | Maintenance.aspx.cs, ProcBuilder.cs        | Admin role required; update/delete operations                          | Admin                     |

---

## 3. Integration Touchpoints & Interface Diagrams

### 3.1 Oracle Database Integration
- **External System:** Oracle DB (PACER_PKG, PACER_EXPORT, PACER_FILE_UPLOAD)
- **Purpose:** All business data, workflow, audit, and validation operations
- **Data Exchanged:** Requests, approvals, user/session, styles/SKUs, logs, exports
- **Protocol/Technology:** Oracle ManagedDataAccess (.NET), Stored Procedures
- **Main Classes/Files:** PCRDAL.cs, ProcBuilder.cs, MySession.cs

```mermaid
sequenceDiagram
    participant .NET_App
    participant Oracle_DB
    .NET_App->>Oracle_DB: ExecuteProc/GetProcDataSet (Stored Procedures)
    Oracle_DB-->>.NET_App: DataTable/DataSet (Results)
```

### 3.2 SMTP Email Notification
- **External System:** SMTP Email Server
- **Purpose:** Send workflow notifications (approvals, declines, deadlines)
- **Data Exchanged:** Email messages (HTML), recipient lists
- **Protocol/Technology:** SMTP (System.Net.Mail or similar)
- **Main Classes/Files:** Email.cs, EmailRevised.cs

```mermaid
sequenceDiagram
    participant .NET_App
    participant SMTP_Server
    .NET_App->>SMTP_Server: Send email (message, recipients)
    SMTP_Server-->>.NET_App: Delivery status
```

### 3.3 File System Integration (Bulk Uploads, Exports)
- **External System:** File System (App_Data/file_upload, Excel exports)
- **Purpose:** Upload bulk SKU/style data, export reports
- **Data Exchanged:** CSV/XLSX files (input/output)
- **Protocol/Technology:** File I/O (.NET System.IO)
- **Main Classes/Files:** Upload.cs, ProcBuilder.cs, GridViewExport.cs

```mermaid
sequenceDiagram
    participant .NET_App
    participant File_System
    .NET_App->>File_System: Read/Write CSV/XLSX files
    File_System-->>.NET_App: File contents/status
```

---

## 4. Appendix

### List of Analyzed Files
- Global.asax.cs
- App_Code/ProcBuilder.cs
- App_Code/PCRDAL.cs
- App_Code/Email.cs
- App_Code/Upload.cs
- App_Code/MySession.cs
- PricingApproval.aspx.cs
- (Other workflow pages: Review.aspx.cs, Edit.aspx.cs, Maintenance.aspx.cs, etc.)

### Glossary of Business Terms

| Term                | Definition                                                                 |
|---------------------|----------------------------------------------------------------------------|
| PACER               | Pricing Approval and Change Event Request system                            |
| DMM                 | Divisional Merchandise Manager (approver role)                              |
| PRICING_SUPER_USER  | User role with pricing approval privileges                                  |
| SKU                 | Stock Keeping Unit (inventory item)                                        |
| Style               | Product style identifier                                                    |
| Exception Price     | Non-standard price request                                                  |
| Original Price      | Standard/original price request                                             |
| Bulk Approval       | Approving multiple requests in one action                                   |
| Dashboard           | Main UI for viewing/filtering requests                                      |
| Audit Log           | Record of user actions/events                                               |
| Visibility User     | User with access to specific pricing events                                 |
| Chain               | Store chain identifier                                                      |
| Effective Date      | Date when price change takes effect                                         |


