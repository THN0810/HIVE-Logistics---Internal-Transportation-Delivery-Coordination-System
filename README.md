# HIVE-Logistics---Internal-Transportation-Delivery-Coordination-System
A BA case study to improve delivery scheduling at HIVE Logistics. I mapped workflows using BPMN/DFD, defined Use Cases and system requirements, and built a simple C#/SQL demo to test the design.
# HIVE Logistics - Internal Transportation & Delivery Coordination System

![Academic Project](https://img.shields.io/badge/Project-Academic-blue?style=flat-square)
![UFM](https://img.shields.io/badge/Institution-UFM-green?style=flat-square)
![C# WinForms](https://img.shields.io/badge/Platform-C%23_WinForms-purple?style=flat-square)
![SQL Server](https://img.shields.io/badge/Database-SQL_Server-red?style=flat-square)

> *This repository showcases my Systems Analysis & Design (SAD) and Database Design capabilities, targeting Business Analyst (BA) methodologies applied to a logistics and supply chain context. Completed as part of the Professional Practice coursework.*

**Author:** Student of Management Information Systems (MIS)  
**Academic Advisor:** MSc. Lam Hoang Truc Mai  
**Target Role:** Business Analyst Intern / Product Owner Intern  

---

## Project Context & Problem Statement

In this academic case study, **HIVE Transportation Services Trading Co., Ltd. (HIVE Logistics)** is analyzed as a medium-sized logistics firm facing operational challenges due to manual workflows.

### The "AS-IS" State (Manual Process)
* **Information Silos:** Booking requests, driver schedules, and vehicle availability are scattered across manual Excel sheets, Zalo groups, and phone logs.
* **Coordination Bottlenecks:** Dispatchers manually match drivers and trucks to shipments, often leading to scheduling conflicts (double-booking) or vehicle capacity mismatches.
* **Lack of Visibility:** Real-time shipment status and trip delays (incidents) are not tracked centrally, delaying accounting, invoicing, and executive reporting.

### The Proposed "TO-BE" State (Centralized System Design)
To address these gaps, this project proposes the **HIVE Logistics System**—a unified desktop-based management system prototype designed to automate dispatch workflows, enforce strict business rules, and streamline billing operations.

---

## BA Methodology & Tools Used

To bridge the gap between business needs and technical design, the following modeling standards and tools were applied:

| Modeling Phase | Tool Used | Deliverables Generated |
| :--- | :--- | :--- |
| **Business Process Modeling** | Draw.io / BPMN 2.0 | To-Be Business Process Diagram mapping cross-functional handoffs. |
| **System Scope & Hierarchy** | Draw.io | Business Function Diagram (BFD) decomposing system modules. |
| **Data Flow Analysis** | Draw.io | Data Flow Diagram (DFD) Context Level & DFD Level 0 tracking data movements. |
| **Functional Requirements** | Enterprise Architect (UML) | 7 Use Case Diagrams & Specifications for 5 system actors. |
| **Database Architecture** | PowerDesigner / SSMS | Conceptual (CDM), Logical (LDM), and Physical (PDM) models in 3NF. |
| **Application Prototyping** | Visual Studio 2022 | C# .NET Windows Forms Desktop Prototype for UI/UX validation. |

---

## Core Business Process (BPMN To-Be)

The proposed system digitizes and automates the logistics lifecycle across 5 distinct swimlanes (Customer, Dispatcher, Driver, Accountant, and Management). 

*(Upload your BPMN diagram to the Images folder and it will display here)*
![BPMN To-Be Process](Images/BPMN_TOBE_HIVE.png)

### Detailed Flow of Information:
1. **Order Intake:** A Customer submits a delivery request. The Dispatcher logs the Transport Order and its detailed cargo specifications into the system.
2. **Automated Resource Validation:** Instead of checking manual logs, the system validates vehicle capacity (`PhuongTien`) and driver availability (`NhanVien`) in real-time.
3. **Dispatch & Notification:** The Dispatcher issues a Dispatch Command. The Driver receives their assigned route, cargo load, and scheduling details directly through their interface.
4. **Execution & Real-Time Tracking:** The Driver updates the shipment status (Loading ➔ Shipping ➔ Delivered). If any issues arise, they file a Logistics Incident Report instantly on-system.
5. **Billing & Invoice Settlement:** Upon successful delivery, the Accountant accesses verified shipping data, issues a Payment Receipt, and automatically generates a detailed VAT Invoicing Record in PDF.
6. **Executive Insights:** The Management accesses read-only performance dashboards to track daily revenue, driver performance, and order-fulfillment rates.

---

## System Modeling & Diagrams

### 1. Business Function Diagram (BFD)
Decomposes the system into 5 main functional modules:
* System Administration & Security (User management, security logs, RBAC)
* Master Data Management (Customer, Driver, Vehicle registers)
* Transportation Coordination (Order creation, resource scheduling, incident tracking)
* Financial Processing (Receipt logging, invoice generation, tax tracking)
* Reporting & Decision Support (Statistical dashboards for executives)

### 2. Data Flow Diagram (DFD)
*(Upload your DFD diagram to the Images folder and it will display here)*
![Data Flow Diagram Context Level](Images/DFD_Context_HIVE.png)

* **Context Diagram:** Establishes the boundary of the HIVE system, illustrating how external agents exchange informational payloads with the core application.
* **DFD Level 0:** Details the internal processes routing data to/from 4 relational data stores:
  * `D1`: Master Accounts & Security
  * `D2`: Transportation & Dispatch Assets
  * `D3`: Order & Delivery Lifecycle
  * `D4`: Financial Receipts & Invoices

### 3. Use Case Diagram & System Actor Matrix
Granular permissions are enforced using Role-Based Access Control (RBAC) across the use cases:

| Actor | Primary Use Cases | Key Actions mapped to UML Diagrams |
| :--- | :--- | :--- |
| **System Admin** | `uc6` | Manages accounts, assigns roles, resets passwords, and audits system access. |
| **Dispatcher** | `uc1`, `uc2`, `uc3`, `uc4` | Registers Customers, manages Driver & Vehicle availability, manages Orders, and schedules Dispatch commands. |
| **Driver** | `uc3 (Extended)` | Receives dispatch commands, accepts/rejects trips, updates delivery logs, and logs road incidents. |
| **Accountant** | `uc5` | Validates completed orders, tracks payment status, and exports shipping invoices. |
| **Manager** | `uc7` | Views consolidated read-only reports on revenue, trip volumes, and fleet efficiency. |

---

## Relational Database Design & Triggers

The database schema `QuanLyDieuPhoiVanChuyen_HIVE` is designed in **Third Normal Form (3NF)** to maintain absolute data integrity and prevent scheduling conflicts using database-level logic.

### Business Rules Enforced by SQL Server Triggers:

**1. Preventing Double-Booking of Drivers & Vehicles**  
To ensure that a driver or a truck is not assigned to overlapping trips simultaneously, a validation trigger is implemented on the `LenhDieuPhoi` (Dispatch Command) table:

```sql
CREATE TRIGGER TRG_LenhDieuPhoi_KhongTrungTaiXe
ON LenhDieuPhoi
FOR INSERT, UPDATE
AS
BEGIN
    -- Check if Driver is already active on another uncompleted trip
    IF EXISTS (
        SELECT 1
        FROM LenhDieuPhoi l
        JOIN inserted i ON l.MaTaiXe = i.MaTaiXe AND l.MaLenh != i.MaLenh
        WHERE l.TrangThaiDVC IN (N'Đã nhận', N'Đang vận chuyển')
    )
    BEGIN
        RAISERROR (N'Error: The assigned driver is currently executing another active shipment.', 16, 1);
        ROLLBACK TRANSACTION;
    END
END;
2. Automatic Invoice Calculations
When a line item is updated, a trigger automatically re-computes the aggregate tax-inclusive total on the parent invoice:
CREATE TRIGGER TRG_ChiTietHDVC_CapNhatTongTienHoaDon
ON ChiTietHDVC
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    UPDATE HoaDonVanChuyen
    SET TongTienTruocThue = (
        SELECT ISNULL(SUM(ThanhTien), 0)
        FROM ChiTietHDVC
        WHERE ChiTietHDVC.MaHDVC = HoaDonVanChuyen.MaHDVC
    )
    WHERE MaHDVC IN (SELECT DISTINCT MaHDVC FROM inserted UNION SELECT DISTINCT MaHDVC FROM deleted);

    -- Recalculate VAT (8%) and final billing total
    UPDATE HoaDonVanChuyen
    SET ThueVAT = TongTienTruocThue * 0.08,
        TongTienSauThue = TongTienTruocThue * 1.08
    WHERE MaHDVC IN (SELECT DISTINCT MaHDVC FROM inserted UNION SELECT DISTINCT MaHDVC FROM deleted);
END;
```
Prototype Demonstration
The prototype is built on C# Windows Forms, simulating the database interactions designed in the SAD phase.
(Upload 1-2 screenshots of your WinForms Application here)
## Local Test Accounts (Role Simulation):
| Username | Password | Enforced Role | BA Validation Goal |
| :--- | :--- | :--- | :--- |
| `admin123` | `(hidden)` | System Administrator | Verify account creation, role assignments, and audit trail logs. |
| `dieuphoivien123` | `(hidden)` | Dispatcher | Test order placement, real-time vehicle mapping, and dispatch. |
| `taixe123` | `(hidden)` | Driver | Test task reception interface, status updates, and incident reporting. |
| `ketoan123` | `(hidden)` | Accountant | Test financial accounting, invoice calculations, and PDF generation. |
| `quanly123` | `(hidden)` | General Manager | Test decision support through dynamic dashboard charts and logs. |
---
## Key Learning Outcomes as a BA Intern
- By executing this academic project, I have gained hands-on experience in:
- Requirements Gathering & Domain Analysis: Transforming raw qualitative business problems from the logistics industry into structured technical       blueprints.
- Cross-Functional Communication: Using standard diagrams (BPMN, UML, DFD) as a common language to align operational, financial, and technical         stakeholders.
- Database & Systems Architecture: Learning how logical business rules (e.g., scheduling conflicts) are systematically enforced through physical       data constraints and relational triggers.
- Agile Prototyping: Validating user interfaces and functional logic by translating abstract use cases into a working C# application.
