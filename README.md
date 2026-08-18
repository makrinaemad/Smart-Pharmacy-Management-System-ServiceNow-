# 💊 CurVia — Smart Pharmacy Management System

**A fully automated pharmacy management platform built on ServiceNow**
---

## 📋 Table of Contents

- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
- [Features](#-features)
- [System Architecture](#-system-architecture)
- [Data Model](#-data-model)
- [Implementation Phases](#-implementation-phases)
- [Technical Components](#-technical-components)
- [Service Portal](#-service-portal)
- [Security & ACLs](#-security--acls)
- [Reports & Dashboard](#-reports--dashboard)
- [Scenarios](#-scenarios)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Installation](#-installation)
- [Author](#-author)

---

## 🌟 Overview

**CurVia** is a fully integrated Smart Pharmacy Management System built on the **ServiceNow** platform. It digitizes and automates all pharmacy operations — from medicine inventory and customer requests to complaint management, SLA enforcement, and real-time reporting.

> Built as an ITI Graduation Project (2026) demonstrating real-world ServiceNow development across 11 implementation phases.

---

## ❗ Problem Statement

Traditional pharmacy operations suffer from:

- ❌ Manual inventory tracking leading to stockouts and expired medicine reaching patients
- ❌ No systematic tracking of customer requests causing poor service quality
- ❌ Manual SLA tracking making it impossible to enforce service level agreements
- ❌ No centralized complaint management leaving customer issues unresolved
- ❌ Absence of real-time dashboards preventing data-driven decisions
- ❌ Common complaints requiring repetitive manual processing

---

## ✅ Features

### 👤 Customer
| Feature | Description |
|---|---|
| Self-Registration | Profile creation via Service Portal |
| Medicine Request | Online request with real-time availability check |
| Request Tracking | Live stage tracking (New → Under Review → Approved → Ready → Completed) |
| Delivery Confirmation | Approve delivery directly from portal |
| Complaint Submission | File complaints with type classification |
| My Requests | Personal request history and approval status |

### 🏥 Admin / Pharmacist
| Feature | Description |
|---|---|
| Inventory Management | Full CRUD on medicine records with expiry and batch tracking |
| Request Approval | Approve or reject with multi-stage workflow |
| Low Stock Alerts | Automated detection + reorder creation with priority scoring |
| Expiry Monitoring | Daily job classifying medicines as Active / Near Expiry / Expired |
| Complaint Management | Auto-assignment + replacement order automation |
| SLA Monitoring | 75% warning + 100% breach notifications |
| Dashboard | Real-time KPIs and operational reports |

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   SERVICE PORTAL                        │
│  Login · Registration · Medicine Request · Complaints   │
│  My Requests · My Approvals · My Reports                │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                 FLOW DESIGNER                           │
│  CurVia_MedicineRequest · Wrong Medicine Flow           │
│  SLA Flows · Registration Flow · Completion SubFlow     │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│              BUSINESS RULES & SCRIPTS                   │
│  Stock Deduction · Expiry Block · Low Stock Alert       │
│  Auto-Assign · Replacement Request · State Sync         │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                   DATA MODEL                            │
│  Pharmacy Medicine · Customer Profile                   │
│  Customer Complaint · Pharmacy Reorder · sc_req_item    │
└─────────────────────────────────────────────────────────┘
```

---

## 🗄️ Data Model

### Custom Tables

| Table | Label | Extends | Purpose |
|---|---|---|---|
| `x_1970979_pharma_0_pharmacy_medicine` | Pharmacy Medicine | — | Medicine inventory |
| `x_1970979_pharma_0_customer_profile` | Customer Profile | — | Patient profiles |
| `x_1970979_pharma_0_customer_complaint` | Customer Complaint | Task | Complaint lifecycle |
| `x_1970979_pharma_0_pharmacy_reorder` | Pharmacy Reorder | Task | Reorder management |

### OOB Tables Extended

| Table | Usage | Custom Fields |
|---|---|---|
| `sc_req_item` | Medicine requests | medicine (ref), customer (ref), replacement_request (bool) |
| `task_sla` | SLA tracking | OOB only |
| `sys_user` | Customer accounts | Linked via Customer Profile |

### Table Relationships

```
Pharmacy Medicine ◄──────── sc_req_item (RITM)
       │                         │
       │                         │
       ▼                         ▼
Pharmacy Reorder        Customer Complaint
                               │
                               ▼
                        Replacement RITM
```

---

## 🚀 Implementation Phases

| Phase | Component | Key Deliverable |
|---|---|---|
| **1** | Scoped App & Tables | Custom tables, form layouts, list views |
| **2** | Service Catalog | Catalog item with availability checker & price auto-fill |
| **3** | Approval Flow | Multi-branch Flow Designer with delivery confirmation |
| **4** | Low Stock Automation | Business Rule + Event + Email pipeline |
| **5** | Expiry Monitoring | Scheduled Job + Approval Block + UI Policy |
| **6** | SLA Management | 2 SLA Definitions + 4 notifications (75% + 100%) |
| **7** | Complaint Management | Auto-assign + UI Policy + Replacement automation |
| **8** | Email Notifications | 11 notifications with branded HTML template |
| **9** | Dashboard & Reports | 3 reports + 4 KPI widgets |
| **10** | Service Portal | Custom widgets + login + registration |
| **11** | ACLs & Security | 36 access control rules |

---

## ⚙️ Technical Components

### Business Rules (9 Active + 2 Global)

| Rule | Table | When | Purpose |
|---|---|---|---|
| Set Customer Name | sc_req_item | Before Insert | Auto-link customer profile to RITM |
| Map Catalog Variables | sc_req_item | Before Insert | Map medicine reference + delivery date |
| Medicine State Sync | sc_req_item | Before Insert/Update | Sync stage with OOB state field |
| Stock Deduction | sc_req_item | Before Update | Deduct stock on Ready for Pickup |
| Stock Restoration | sc_req_item | Before Update | Restore stock on cancellation |
| Low Stock Alert | pharmacy_medicine | After Update | Fire event + create reorder |
| Generate Medicine Batch | pharmacy_reorder | After Update | Create new batch on reorder completion |
| Auto-Assign Complaint | customer_complaint | Before Insert | Assign to Pharmacy Admin Group |
| Map Request to Complaint | customer_complaint | Before Insert | Copy medicine + customer refs |
| Block Expired Approval ⭐ | sysapproval_approver | Before Update | Block expired medicine approvals |
| Create Replacement ⭐ | customer_complaint | After Update | Auto-create replacement via Cart API |

> ⭐ Global scope rules

### Flows (Active)

```
CurVia_MedicineRequest          ← Main approval lifecycle
├── Catalog Task For The Request ← Pharmacist preparation sub-flow
└── Medicine Request Completed   ← Completion/cancellation sub-flow

Wrong Medicine Flow              ← Complaint approval + replacement
Pharmacy Customer Registration   ← Self-registration processing
Medicine Request SLA flow        ← 75% warning + breach alerts
Pharmacy Complain SLA flow       ← Complaint SLA notifications
```

### Script Include

```javascript
// MedicineAvailability — Client Callable (GlideAjax)
// Checks real-time medicine stock status from Service Portal
// Returns: { status: 'available' | 'low' | 'expired' | 'not_found', quantity: N }
```

### Catalog Client Scripts

| Script | Variable | Purpose |
|---|---|---|
| Medicine Availability Checker | medicine_name | Real-time stock check on portal |
| Pickup Date Validation | preferred_pickup_date | Enforce 1-hour minimum lead time |
| Complaint Type Mandatory | complaint_type | Force description when type = Other |

### UI Policies

| Policy | Table | Condition | Action |
|---|---|---|---|
| Expired Medicine Read-Only | pharmacy_medicine | status = expired | All fields → read-only |
| Medicine Name Mandatory | pharmacy_medicine | Always | medicine_name, category → mandatory |
| Resolution Notes | customer_complaint | state = resolved/closed | resolution_notes → visible + mandatory |
| Replacement Fields | customer_complaint | approval = approved + wrong_medicine | replacement fields → visible |

### SLA Definitions

| SLA | Table | Duration | Schedule | Start | Stop |
|---|---|---|---|---|---|
| MedicineRequestSLA | sc_req_item | 2 Hours | 8×5 weekdays | stage = under_review | stage = approved/rejected |
| PharmacyComplaintSLA | customer_complaint | 2 Hours | 8×5 weekdays | state = New | state = Closed Complete |

### Notifications (11 Active)

| # | Name | Trigger | Recipient |
|---|---|---|---|
| N-01 | Low Stock Alert | pharmacy.low_stock event | Admin Group |
| N-02 | MedicineRequestSLA Warning | 75% elapsed | Admin Group |
| N-03 | MedicineRequestSLA Breach | has_breached = true | Admin Group |
| N-04 | ComplaintSLA Warning | 75% elapsed | Admin Group |
| N-05 | ComplaintSLA Breach | has_breached = true | Admin Group |
| N-06 | Medicine Ready — Customer | stage = ready_for_pickup | Customer |
| N-07 | Request Rejected — Customer | stage = rejected | Customer |
| N-08 | Request Cancelled — Admin | stage = canceled | Admin |
| N-09 | New Complaint — Admin | complaint created | Admin Group |
| N-10 | Complaint Approved — Customer | complaint approved | Customer |
| N-11 | Complaint Rejected — Customer | complaint rejected | Customer |

---

## 🖥️ Service Portal

### Pages

| Page | URL | Audience |
|---|---|---|
| Home | `/curvia?id=curvia` | All users |
| Login | `/curvia?id=login` | Unauthenticated |
| Registration | `/curvia?id=sn_user_registration` | New patients |
| Medicine Request | `/curvia?id=sc_cat_item` | Logged-in customers |
| My Requests | `/curvia?id=my_requests` | Logged-in customers |
| My Approvals | `/curvia?id=my_approvals` | Logged-in customers |
| My Reports | `/curvia?id=my_reports` | Logged-in customers |

### Custom Widgets

| Widget | Description |
|---|---|
| Pharmacy Login | Branded login with forgot password + register link |
| Pharmacy Welcome | Animated banner with floating bubbles + personalized greeting |
| Pharmacy Hero Banner | Full-width cover with quick action links |

---

## 🔐 Security & ACLs

### Roles

| Role | Permissions |
|---|---|
| `x_1970979_pharma_0.admin` | Full CRUD, approvals, reports, dashboard |
| `x_1970979_pharma_0.pharmacist` | Inventory read/write, view complaints, complete tasks |
| `x_1970979_pharma_0.customer` | Own requests, own complaints, portal access only |

### ACL Summary

```
36 Total ACL Rules covering:
├── x_1970979_pharma_0_pharmacy_medicine     → CRUD by role
├── x_1970979_pharma_0_customer_profile      → Own record for customer
├── x_1970979_pharma_0_customer_complaint    → Own record for customer
├── x_1970979_pharma_0_pharmacy_reorder      → Admin + Pharmacist only
└── MedicineAvailability (Script Include)    → Execute: all authenticated
```

---

## 📊 Reports & Dashboard

### Reports

| Report | Type | Source | Purpose |
|---|---|---|---|
| Inventory by Status | Pie Chart | pharmacy_medicine | Active / Near Expiry / Expired breakdown |
| Requests by Stage | Bar Chart | sc_req_item | Pipeline bottleneck visibility |
| Top Medicines | Bar Chart | sc_req_item | Demand analysis (Top 10) |

### Dashboard KPI Widgets

| Widget | Value | Color |
|---|---|---|
| Active Medicines | Count where status = active | 🟢 Green |
| Near Expiry | Count where status = near_expiry | 🟠 Orange |
| Expired Medicines | Count where status = expired | 🔴 Red |
| Low Stock Count | Count where low_stock = true | 🟠 Orange |

---

## 🎬 Scenarios

### Happy Path
```
Customer registers → Submits medicine request → Admin approves
→ Pharmacist prepares → Customer confirms delivery → Completed ✅
```

### Edge Cases Handled
```
❌ Expired medicine approval attempt     → Blocked by BR (setAbortAction)
❌ Insufficient stock at pickup          → Blocked by BR (addErrorMessage)
❌ Customer doesn't confirm delivery     → Auto-cancelled after timeout
❌ Wrong medicine complaint              → Replacement auto-created via Cart API
⚠️ Low stock detected                   → Reorder created + admin alerted
⚠️ Medicine expiring in 30 days         → Status → Near Expiry
🕐 SLA at 75%                           → Warning email sent
🚨 SLA at 100%                          → Breach email sent
```

---

## 🛠️ Tech Stack

```
Platform        ServiceNow (PDI)
Scope           x_1970979_pharma_0
Version         1.0.0

Server Side     GlideRecord · GlideAjax · GlideDateTime
                Cart API · gs.eventQueue · AbstractAjaxProcessor

Client Side     AngularJS · g_form API · GlideAjax

Automation      Flow Designer · Business Rules · Scheduled Jobs
                Data Policies · UI Policies · Client Scripts

Security        ACL (36 rules) · Scoped Application · Role-based access

Portal          Service Portal · Custom Widgets · Record Producers
                Catalog Items · Stage Sets

Monitoring      SLA Definitions · task_sla · Email Notifications
                Event Registry · Reports · Dashboard
```

---

## 📁 Project Structure

```
CurVia/
├── 📄 README.md
├── 📦 update_sets/
│   ├── sys_remote_update_set_main.xml       ← Main update set
│   └── sys_remote_update_set_global.xml     ← Global scope rules
├── 📚 documentation/
│   ├── CurVia_Full_Documentation.pdf        ← Complete technical docs
├── 🖼️ screenshots/
│   ├── portal/
│   │   ├── login.png
│   │   ├── welcome.png
│   │   ├── medicine_request.png
│   │   ├── my_requests.png
│   │   └── my_approvals.png
│   ├── flows/
│   │   ├── main_approval_flow.png
│   │   ├── wrong_medicine_flow.png
│   │   └── sla_flows.png
│   ├── forms/
│   │   ├── medicine_form.png
│   │   ├── complaint_form.png
│   │   └── reorder_form.png
│   └── dashboard/
│       └── pharmacy_dashboard.png
└── 🎨 assets/
    └── curvia_logo.png
```

---

## 📥 Installation

### Prerequisites
- ServiceNow Personal Developer Instance (PDI)
- Admin access to the instance

### Steps

**1. Import Update Sets**
```
All → System Update Sets → Retrieved Update Sets → Import XML
Upload: sys_remote_update_set_main.xml
Upload: sys_remote_update_set_global.xml
```

**2. Preview & Commit**
```
Open each update set → Preview → Commit
(Resolve any conflicts if prompted)
```

**3. Create Test Users**
```
System Security → Users → New

pharmacy.admin   → roles: x_1970979_pharma_0.admin + itil
test.pharmacist  → roles: x_1970979_pharma_0.pharmacist + itil
test.customer    → roles: x_1970979_pharma_0.customer
```

**4. Add Users to Groups**
```
Pharmacy Admin Group    → add pharmacy.admin
Pharmacy Pharmacist Group → add test.pharmacist
Pharmacy Customer Group → add test.customer
```

**5. Access the Portal**
```
https://YOUR_INSTANCE.service-now.com/curvia
```

---

## 👩‍💻 Author

**Makrina Emad Petro**
ITI Graduate — Information Systems
ServiceNow Developer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin)]([https://linkedin.com/in/YOUR_PROFILE](https://www.linkedin.com/in/makrina-petro-95a95523b))
