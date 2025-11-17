# High-Level Architecture Document

## 1. Overview
- **Purpose:**  
  This document provides a comprehensive high-level architectural analysis of the PACER .NET Framework application. It details the system's layers, components, technology stack, integration points, deployment topology, and major design patterns, serving as a blueprint for technical stakeholders.
- **Scope:**  
  The analysis covers all relevant files and folders, including entry points (Global.asax), code-behind, business/data/integration logic (App_Code), configuration, UI assets, and deployment/configuration artifacts. All statements are backed by direct evidence from the codebase.
- **Audience:**  
  Architects, backend/frontend developers, DevOps, security reviewers, maintainers, and modernization/cloud migration teams.

## 2. Technology Stack
| Layer/Component | Technology/Framework           | Version      | Notes                                                                                   |
|-----------------|-------------------------------|--------------|-----------------------------------------------------------------------------------------|
| .NET Framework  | .NET Framework                | 4.8          | TargetFrameworkVersion in PCR.csproj                                                    |
| Language        | C#                            |              | All code-behind and App_Code files are C#                                               |
| Web Framework   | ASP.NET Web Forms             |              | Web Forms (ASPX, code-behind, Global.asax)                                              |
| ORM/Data Access | Oracle.ManagedDataAccess       | 19.10.1      | NuGet, referenced in PCR.csproj, used in App_Code/PCRDAL.cs, ProcBuilder.cs             |
| Frontend        | ASPX, JavaScript, jQuery, Bootstrap, DataTables | Bootstrap 4.x, jQuery 3.5.1 | Scripts and Content folders, Site.Master, DataTables CSS/JS                             |
| Database        | Oracle                        |              | All data access via Oracle stored procedures                                            |
| Messaging       | SMTP                          |              | Email notifications via App_Code/Email.cs                                               |
| Caching         | None explicit                 |              | Not observed in code or config                                                          |
| Logging         | Custom (Oracle SP)            |              | All logs written to Oracle via ProcBuilder.WriteToLog                                   |
| Others          | BouncyCastle, CryptoWrapper, NPOI, GenericParser | BouncyCastle 1.8.6, NPOI 2.5.3 | For encryption, Excel export, CSV parsing                                               |

## 3. Architectural Layers & Components

- **Layered Diagram:**  
```mermaid
graph TD
subgraph Presentation Layer
    UI_ASPX[ASPX Pages (.aspx, .master)]
    JS[JavaScript/jQuery/Bootstrap]
    UI_ASPX --> JS
end

subgraph Business Logic Layer
    BL_ProcBuilder[ProcBuilder.cs]
    BL_Pacer[Pacer.cs, AddPacer.cs]
    BL_Workflows[Approval, Validation, Notification]
end

subgraph Data Access Layer
    DAL_PCRDAL[PCRDAL.cs - Oracle DB Access]
end

subgraph Integration Layer
    INT_Email[Email.cs, SMTP Server]
    INT_FileUpload[Upload.cs, File System]
    INT_Encryption[BouncyCastle, CryptoWrapper]
end

subgraph Infrastructure
    INF_Session[MySession.cs]
    INF_Logging[ProcBuilder.WriteToLog]
    INF_Config[Web.config, App.config]
end

UI_ASPX --> BL_ProcBuilder
JS --> BL_Workflows
BL_ProcBuilder --> DAL_PCRDAL
DAL_PCRDAL --> OracleDB[(Oracle Database)]
BL_ProcBuilder --> INT_Email
BL_ProcBuilder --> INT_FileUpload
INT_Encryption --> DAL_PCRDAL
INF_Session --> BL_ProcBuilder
INF_Logging --> BL_ProcBuilder
```

- **Layer Descriptions:**  
  - **Presentation Layer:**  
    - ASPX pages (Default.aspx, Request.aspx, Upload.aspx, PricingDashboard.aspx, PricingApproval.aspx, Maintenance.aspx, Edit.aspx, Review.aspx, Search.aspx, Calendar.aspx)
    - Site.Master for layout, Content and Scripts folders for Bootstrap, jQuery, DataTables, and custom JS/CSS
  - **Business Logic Layer:**  
    - App_Code/ProcBuilder.cs: Orchestrates workflows, validation, approval, logging, and business rules
    - App_Code/AddPacer.cs, Pacer.cs: Request creation and detail management
    - App_Code/Style.cs, SKU.cs, Store.cs, EffectiveDate.cs: Domain logic for styles, SKUs, stores, dates
    - App_Code/MySession.cs: User/session/role management
  - **Data Access Layer:**  
    - App_Code/PCRDAL.cs: Centralized Oracle DB access, command execution, and SQL helpers
    - App_Code/QueryBuilder.cs: Bulk insert/query logic for uploads
  - **Integration Layer:**  
    - App_Code/Email.cs: SMTP email notification logic
    - App_Code/Upload.cs: File system interaction for bulk uploads
    - BouncyCastle, CryptoWrapper: Encryption (referenced in csproj)
  - **Infrastructure:**  
    - Logging via ProcBuilder.WriteToLog (Oracle SP merch_user.user_log)
    - Configuration via Web.config (not readable, but referenced in code)
    - Session/user context via MySession.cs

- **Major Components Table:**
| Component Name         | Role/Purpose                                  | Layer              | Key Dependencies                | Location (Path)                             |
|-----------------------|-----------------------------------------------|--------------------|----------------------------------|---------------------------------------------|
| Global.asax.cs        | App entry point, session/app event hooks      | Infrastructure     | ProcBuilder, MySession           | /Global.asax.cs                            |
| Request.aspx(.cs)     | Request creation UI/logic                     | Presentation/BL    | ProcBuilder, AddPacer, MySession | /Request.aspx, /Request.aspx.cs             |
| Upload.aspx(.cs)      | Bulk upload UI/logic                          | Presentation/BL    | QueryBuilder, ProcBuilder        | /Upload.aspx, /Upload.aspx.cs               |
| PricingApproval.aspx(.cs) | Approval workflow UI/logic                | Presentation/BL    | ProcBuilder, Email               | /PricingApproval.aspx, /PricingApproval.aspx.cs |
| PricingDashboard.aspx(.cs) | Dashboard/reporting UI/logic             | Presentation/BL    | ProcBuilder, GridViewExport      | /PricingDashboard.aspx, /PricingDashboard.aspx.cs |
| App_Code/ProcBuilder.cs | Business workflow, validation, logging      | Business Logic     | PCRDAL, Oracle, MySession        | /App_Code/ProcBuilder.cs                    |
| App_Code/PCRDAL.cs    | Oracle DB access abstraction                  | Data Access        | Oracle.ManagedDataAccess         | /App_Code/PCRDAL.cs                         |
| App_Code/Email.cs     | Email notification logic                      | Integration        | SMTP, System.Net.Mail            | /App_Code/Email.cs                          |
| App_Code/QueryBuilder.cs | Bulk SQL for uploads                       | Data Access        | PCRDAL, Oracle                   | /App_Code/QueryBuilder.cs                   |
| App_Code/AddPacer.cs  | Adds request headers/details                  | Business Logic     | ProcBuilder, PCRDAL              | /App_Code/AddPacer.cs                       |
| App_Code/MySession.cs | User/session/role management                  | Infrastructure     | HttpContext, FormsAuth           | /App_Code/MySession.cs                      |
| App_Code/Upload.cs    | File system upload helpers                    | Integration        | System.IO, FileSystem            | /App_Code/Upload.cs                         |
| App_Code/GridViewExport.cs | Export to Excel/CSV                      | Presentation/BL    | NPOI, System.Web.UI.WebControls  | /App_Code/GridViewExport.cs                 |
| App_Code/Style.cs, SKU.cs, Store.cs | Domain logic for styles/SKUs/stores | Business Logic | PCRDAL, Oracle                   | /App_Code/Style.cs, /App_Code/SKU.cs, /App_Code/Store.cs |

## 4. Integration Points

| Integration Name         | Type (API/DB/etc.) | Technology/Protocol      | Purpose                                   | Security/Auth         | Location in Code                |
|-------------------------|--------------------|--------------------------|-------------------------------------------|-----------------------|----------------------------------|
| Oracle Database         | Database           | Oracle.ManagedDataAccess | All business data, workflow, audit, logs  | DB user/connection    | App_Code/PCRDAL.cs, ProcBuilder.cs |
| SMTP Email              | Messaging          | SMTP/.NET Mail           | Workflow notifications, errors            | SMTP relay            | App_Code/Email.cs                |
| File System (Uploads)   | File System        | System.IO                | Bulk CSV upload, temp file storage        | NTFS ACLs             | App_Code/Upload.cs, Upload.aspx.cs|
| Excel Export            | File Download      | NPOI, HTTP               | Data/report export to Excel/CSV           | N/A                   | App_Code/GridViewExport.cs       |
| Encryption              | Library            | BouncyCastle, CryptoWrapper | Data encryption (if used)               | N/A                   | Referenced in csproj             |

## 5. Deployment & Infrastructure
- **Hosting Model:**  
  IIS (UseIIS true in csproj), ASP.NET Web Forms, Windows Authentication enabled (IISExpressWindowsAuthentication)
- **Deployment Topology Diagram:**  
```mermaid
%% Deployment diagram
graph TD
    User --> IIS
    IIS --> PACER_App[.NET Web Forms App]
    PACER_App --> OracleDB[(Oracle Database)]
    PACER_App --> SMTP[SMTP Server]
    PACER_App --> FileSys[File System (Uploads)]
```
- **Environments:**  
  Web.Debug.config, Web.Release.config present for environment-specific settings (connection strings, etc.)
- **Scalability/Availability:**  
  Not explicitly defined, but IIS hosting supports web farm/load balancing. Oracle DB is central point.
- **Network/Security:**  
  Windows Authentication (IIS), likely internal enterprise deployment. File system access for uploads. SMTP relay for emails.

## 6. Architectural Patterns & Design Decisions
- **Patterns Used:**  
  - Layered Architecture: Clear separation of Presentation, Business Logic, Data Access, Integration, Infrastructure
  - Repository/DAO: PCRDAL abstracts DB access
  - Procedural Workflow: ProcBuilder orchestrates business flows via stored procs
  - Code-Behind: ASP.NET Web Forms with code-behind for UI logic
  - Custom Logging: All logs/audits via Oracle stored procedures
- **Key Decisions:**  
  - Heavy reliance on Oracle stored procedures for business logic, validation, and workflow
  - All exceptions and user actions logged centrally in Oracle
  - Role-based access and session management via MySession
  - Bulk operations (upload, export) handled via file system and batch DB calls
- **Cross-Cutting Concerns:**  
  - Logging (ProcBuilder.WriteToLog)
  - Error handling (try/catch, logs to Oracle)
  - Security (Windows Auth, session/role checks)
  - Email notifications (workflow status, errors)
  - Data validation (DB-level and code-level)

## 7. Appendix

- **File/Folder Inventory:**  
  - **Entry Points:**  
    - /Global.asax, /Global.asax.cs
    - /Default.aspx(.cs), /Request.aspx(.cs), /Upload.aspx(.cs), /PricingDashboard.aspx(.cs), /PricingApproval.aspx(.cs), /Maintenance.aspx(.cs), /Edit.aspx(.cs), /Review.aspx(.cs), /Search.aspx(.cs), /Calendar.aspx(.cs)
  - **Business/Data/Integration Logic:**  
    - /App_Code/ProcBuilder.cs, /App_Code/PCRDAL.cs, /App_Code/Email.cs, /App_Code/QueryBuilder.cs, /App_Code/AddPacer.cs, /App_Code/MySession.cs, /App_Code/Style.cs, /App_Code/SKU.cs, /App_Code/Store.cs, /App_Code/EffectiveDate.cs, /App_Code/Upload.cs, /App_Code/GridViewExport.cs, /App_Code/Pacer.cs, /App_Code/EmailRevised.cs, /App_Code/StyleColors.cs, /App_Code/PRAEmail.cs
  - **Configuration:**  
    - /PCR.csproj, /Web.config, /Web.Debug.config, /Web.Release.config, /packages.config
  - **UI/Assets:**  
    - /Content (Bootstrap, DataTables CSS), /Scripts (jQuery, Bootstrap, DataTables JS), /Images, /CSS
    - /App_Data (uploaded files, temp data)
  - **Other:**  
    - /Properties/AssemblyInfo.cs, /PACER Documentation.docx, /PACER Help Doc.doc, various Excel templates

- **Glossary:**
  - **PACER:** Pricing and Approval Change Event Request (core business object)
  - **DMM:** Divisional Merchandise Manager (approver role)
  - **SKU:** Stock Keeping Unit (product identifier)
  - **Style:** Product style identifier
  - **Chain/Store:** Organizational hierarchy for pricing events
  - **Original/Exception:** Types of pricing requests
  - **MPU:** Mass Price Update (bulk operation)
  - **Effective Date:** Date when pricing change takes effect
  - **Approval Workflow:** Multi-step process for request validation and sign-off
  - **Bulk Upload:** Batch import of pricing data via CSV
  - **Audit Log:** Record of user actions and errors for compliance/tracing

---

*End of High-Level Architecture Document*
