# High-Level Architecture Document

## 1. Overview
- **Purpose:**  
  This document provides a comprehensive high-level architectural analysis of the PACER (Pricing Approval and Exception Request) .NET Framework application. It details the system's structure, technology stack, major components, integration points, deployment topology, and design patterns to support onboarding, modernization, maintenance, and security reviews.
- **Scope:**  
  The analysis covers all application layers, key business logic, data access, integration with external systems, and deployment configuration, based on explicit evidence from the codebase.
- **Audience:**  
  Intended for solution architects, backend developers, DevOps engineers, security reviewers, and technical managers involved in the maintenance, enhancement, or migration of the PACER application.

---

## 2. Technology Stack
| Layer/Component | Technology/Framework              | Version     | Notes                                            |
|-----------------|----------------------------------|-------------|--------------------------------------------------|
| .NET Framework  | .NET Framework                   | 4.8         | From PCR.csproj                                  |
| Language        | C#                               |             | All code files are C#                            |
| Web Framework   | ASP.NET Web Forms                |             | .aspx pages, code-behind, Global.asax            |
| ORM/Data Access | ADO.NET (Oracle.ManagedDataAccess)| 19.10.1     | Direct SQL, stored procedures, Oracle DB         |
| Frontend        | HTML, JavaScript, jQuery, DataTables, Bootstrap |         | No SPA; classic Web Forms with JS enhancements   |
| Database        | Oracle                           | (19c+)      | Oracle.ManagedDataAccess, connection strings      |
| Messaging       | Email (SMTP)                     |             | Workflow/notification via Email.cs               |
| Caching         | None explicit                    |             | No evidence of caching framework                 |
| Logging         | Custom (ProcBuilder.WriteToLog)  |             | No log4net/Serilog; custom log calls             |
| Others          | NPOI, BouncyCastle, SharpZipLib, GenericParsing | See .csproj | For Excel, crypto, ZIP, CSV, etc.                |

---

## 3. Architectural Layers & Components

- **Layered Diagram:**  
  ```mermaid
  graph TD
    UI["Presentation Layer (ASPX Pages)"]
    BL["Business Logic Layer (App_Code)"]
    DAL["Data Access Layer (PCRDAL)"]
    DB["(Oracle Database)"]
    EXT["External Services (Email, File Upload)"]
    UI --> BL
    BL --> DAL
    DAL --> DB
    BL --> EXT
  ```

- **Layer Descriptions:**  
  - **Presentation Layer:**  
    - ASP.NET Web Forms (.aspx pages, Site.Master)
    - Handles user interaction, data display, and input forms
    - Main folders/files: Default.aspx, Edit.aspx, Maintenance.aspx, PricingDashboard.aspx, PricingApproval.aspx, Review.aspx, Search.aspx, Upload.aspx, Calendar.aspx, Site.Master
  - **Business Logic Layer:**  
    - App_Code classes encapsulating domain logic, workflow, and validation
    - Main files: Pacer.cs, Email.cs, AddPacer.cs, EffectiveDate.cs, Style.cs, Store.cs, SKU.cs, QueryBuilder.cs, EmailRevised.cs, PRAEmail.cs, MySession.cs
  - **Data Access Layer:**  
    - Direct ADO.NET access to Oracle via PCRDAL.cs
    - Executes SQL and stored procedures, handles connections and data mapping
  - **Integration Layer:**  
    - Email notifications (Email.cs, PRAEmail.cs)
    - File upload and export (Upload.cs, GridViewExport.cs)
    - External libraries for crypto (BouncyCastle), Excel (NPOI), ZIP (SharpZipLib), CSV (GenericParsing)
  - **Infrastructure:**  
    - Configuration via Web.config (not accessible, but referenced)
    - Session management (MySession.cs)
    - Logging (ProcBuilder.WriteToLog)
    - Deployment via IIS (from .csproj)

- **Major Components Table:**
  | Component Name      | Role/Purpose                                    | Layer                | Key Dependencies                | Location (Path)                          |
  |--------------------|--------------------------------------------------|----------------------|----------------------------------|------------------------------------------|
  | Default.aspx       | Home/dashboard UI                                | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/Default.aspx     |
  | Edit.aspx          | Edit PACER requests                              | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/Edit.aspx        |
  | Maintenance.aspx   | System/data maintenance                          | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/Maintenance.aspx |
  | PricingDashboard.aspx | Pricing dashboard/reports                     | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/PricingDashboard.aspx |
  | PricingApproval.aspx | Approve pricing requests                       | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/PricingApproval.aspx |
  | Review.aspx        | Review PACER requests                            | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/Review.aspx      |
  | Search.aspx        | Search PACER records                             | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/Search.aspx      |
  | Upload.aspx        | File upload (CSV, Excel)                         | Presentation         | App_Code, PCRDAL, NPOI           | /PACER-release-MasterV3/Upload.aspx      |
  | Calendar.aspx      | Calendar view                                    | Presentation         | App_Code, PCRDAL                 | /PACER-release-MasterV3/Calendar.aspx    |
  | Site.Master        | Master page/layout                               | Presentation         | -                                | /PACER-release-MasterV3/Site.Master      |
  | Pacer.cs           | Domain entity/model for PACER requests           | Business Logic       | -                                | /App_Code/Pacer.cs                       |
  | Email.cs           | Workflow/email notification logic                | Business Logic       | PCRDAL, ProcBuilder, MySession   | /App_Code/Email.cs                       |
  | PCRDAL.cs          | Data access (Oracle)                             | Data Access          | Oracle.ManagedDataAccess, Crypto | /App_Code/PCRDAL.cs                      |
  | ProcBuilder.cs     | Stored procedure helpers, logging                | Business/Infra       | PCRDAL, Logging                  | /App_Code/ProcBuilder.cs                 |
  | Upload.cs          | File upload logic                                | Integration          | NPOI, PCRDAL                     | /App_Code/Upload.cs                      |
  | GridViewExport.cs  | Export data to Excel                             | Integration          | NPOI                             | /App_Code/GridViewExport.cs              |
  | MySession.cs       | Custom session management                        | Infrastructure       | System.Web.SessionState          | /App_Code/MySession.cs                   |
  | AddPacer.cs        | Add new PACER requests                           | Business Logic       | PCRDAL, Pacer                    | /App_Code/AddPacer.cs                    |
  | Style.cs, SKU.cs, Store.cs, StyleColors.cs | Domain models            | Business Logic       | -                                | /App_Code/                               |
  | PRAEmail.cs, EmailRevised.cs | Specialized email logic                | Integration          | PCRDAL, ProcBuilder              | /App_Code/                               |

---

## 4. Integration Points

| Integration Name     | Type (API/DB/etc.) | Technology/Protocol     | Purpose                              | Security/Auth            | Location in Code             |
|----------------------|--------------------|------------------------|--------------------------------------|--------------------------|------------------------------|
| Oracle Database      | Database           | Oracle.ManagedDataAccess| Data storage and business logic      | Encrypted credentials    | /App_Code/PCRDAL.cs          |
| Email (SMTP)         | Messaging          | SMTP/.NET SmtpClient   | Workflow notifications               | Not explicit             | /App_Code/Email.cs, PRAEmail.cs |
| File Upload/Export   | File System        | NPOI, SharpZipLib      | Import/export Excel, CSV files       | File system permissions  | /App_Code/Upload.cs, GridViewExport.cs |
| CryptoWrapper        | Encryption         | BouncyCastle           | Decrypt DB credentials, data         | Custom, BouncyCastle     | /App_Code/PCRDAL.cs, .csproj  |

---

## 5. Deployment & Infrastructure

- **Hosting Model:**  
  IIS (Internet Information Services), with support for IIS Express in development (from .csproj)

- **Deployment Topology Diagram:**  
  ```mermaid
  graph TD
    User --> IIS
    IIS --> WebApp[ASP.NET Web Forms App]
    WebApp --> OracleDB[(Oracle Database)]
    WebApp --> SMTP["SMTP Server (Email)"]
    WebApp --> FileSys["File System (Uploads/Exports)"]
  ```

- **Environments:**  
  - Dev/Test/Prod separation is implied by session flags and connection string handling (MySession.Current.LinkToProd, etc.)
  - Web.config (not accessible) likely contains environment-specific settings

- **Scalability/Availability:**  
  - Standard IIS deployment; no explicit evidence of load balancing or high-availability features

- **Network/Security:**  
  - Database credentials are encrypted and decrypted at runtime
  - Windows Authentication enabled for IIS Express (from .csproj)
  - No explicit firewall/DMZ configuration in codebase

---

## 6. Architectural Patterns & Design Decisions

- **Patterns Used:**  
  - Layered architecture (Presentation, Business Logic, Data Access)
  - Domain Model (Pacer, Style, SKU, Store, etc.)
  - Repository-like data access (PCRDAL), but not full Repository pattern
  - Custom session/context management (MySession)
  - Procedural workflow via stored procedures and helper classes

- **Key Decisions:**  
  - Direct ADO.NET for Oracle integration (no ORM)
  - Business logic split between code and Oracle stored procedures
  - Heavy use of App_Code for shared logic
  - Email as primary workflow notification mechanism

- **Cross-Cutting Concerns:**  
  - Logging: Custom log method (ProcBuilder.WriteToLog)
  - Security: Encrypted configuration, Windows Authentication
  - Error Handling: Try/catch blocks in DAL, but no global error handler
  - No explicit caching or dependency injection

---

## 7. Appendix

- **File/Folder Inventory:**
  | File/Folder                      | Architectural Mapping          |
  |----------------------------------|-------------------------------|
  | /Default.aspx, /Edit.aspx, /Maintenance.aspx, /PricingDashboard.aspx, /PricingApproval.aspx, /Review.aspx, /Search.aspx, /Upload.aspx, /Calendar.aspx | Presentation Layer (UI)        |
  | /Site.Master                     | Presentation/Layout           |
  | /App_Code/PCRDAL.cs              | Data Access Layer             |
  | /App_Code/Pacer.cs, /App_Code/Style.cs, /App_Code/SKU.cs, /App_Code/Store.cs, /App_Code/StyleColors.cs | Domain Models                  |
  | /App_Code/Email.cs, /App_Code/PRAEmail.cs, /App_Code/EmailRevised.cs | Integration/Workflow           |
  | /App_Code/ProcBuilder.cs, /App_Code/QueryBuilder.cs | Business Logic/Infrastructure |
  | /App_Code/Upload.cs, /App_Code/GridViewExport.cs | Integration/File Handling      |
  | /App_Code/MySession.cs           | Infrastructure/Session        |
  | /App_Data/                       | Data files, uploads           |
  | /Content/, /CSS/, /Images/       | Static assets (CSS, images)   |
  | /Scripts/                        | Frontend JS, DataTables, Bootstrap |
  | /Global.asax, /Global.asax.cs    | Application entry/lifecycle   |
  | /Web.config, /Web.Debug.config, /Web.Release.config | Configuration                 |
  | /PCR.csproj                      | Project metadata, references  |
  | /packages.config                 | NuGet packages (not accessible) |

- **Glossary:**
  - **PACER:** Pricing Approval and Exception Request system
  - **App_Code:** ASP.NET folder for shared code (business logic, data access)
  - **PCRDAL:** Data Access Layer class for Oracle integration
  - **ADO.NET:** .NET data access technology
  - **NPOI:** .NET library for reading/writing Excel files
  - **SMTP:** Simple Mail Transfer Protocol, used for email
  - **IIS:** Internet Information Services, Windows web server
  - **Master Page:** ASP.NET feature for consistent layout
  - **Session:** User-specific state management in ASP.NET

---
