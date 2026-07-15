# UI/UX Design Prompt: Tax Audit Management System

---

## 1. Project Overview

### 1.1 Project Name
**Tax Audit Management System (TAMS)** – A comprehensive digital platform for managing the end-to-end tax audit lifecycle, from planning and case selection to execution, reporting, and quality assurance.

### 1.2 Project Objective
Design a modern, intuitive, and secure user interface for a tax administration system that digitizes and automates audit processes. The system serves multiple user roles across the tax authority, enabling efficient case management, collaboration, and decision-making.

### 1.3 Key Features
- Annual audit planning and case cascading
- Risk-based case selection and automated assignment
- Multiple audit types: Desk, Comprehensive, Transfer Pricing, Issue, Joint
- Entry and Exit conference management
- Audit execution with data analytics and CAAT
- Reporting, finalization, and assessment notice issuance
- Taxpayer communication portal
- Quality assurance review
- Fraud investigation trigger

---

## 2. User Roles & Their Needs

| **Role** | **Primary Responsibilities** | **Key Pages / Modules** |
| :--- | :--- | :--- |
| **Auditor** | Conduct audits, schedule conferences, record findings, draft reports | Dashboard, My Audits, Entry Conferences, Exit Conferences, Audit Execution Workspace, Draft Reports, Taxpayer Communication |
| **Team Leader** | Approve work, monitor team progress, review reports | Dashboard, Team Audits, Approvals (Entry/Exit Conferences, Draft Reports), Progress Monitoring |
| **Process Owner** | Select/prioritize cases, manage notices, handle objections | Dashboard, Case Selection, Notice Management, Objection Validity Check, Audit Plan Management |
| **Director / Authorized Official** | Approve final reports, issue assessments, account adjustments | Dashboard, Final Approvals, Assessment Issuance, Account Adjustment |
| **Joint Audit Team** | Collaborate across agencies, share documents, run CAAT | Joint Audit Workspace, Shared Document Repository, CAAT Dashboard, Consolidated Reporting |
| **Taxpayer / Tax Agent** | View audit status, confirm meetings, upload documents, raise objections | Taxpayer Portal, Meeting Confirmations, Document Upload, Objection Submission, Communication Hub |
| **QA Team** | Review completed audits, provide feedback, track improvements | QA Dashboard, Case Selection, QA Review Workspace, Follow-up Tracker |
| **Admin Staff** | Handle printing, mailing, undelivered notices | Notice Printing, Bulk Operations, Undelivered Tracking |

---

## 3. Page Inventory by Role

### 3.1 Auditor Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **Main Dashboard** | Centralized view of all tasks | KPI cards (Active Audits, Due This Week, Pending Conferences, Draft Reports); Task list with statuses; Quick Actions; Statute clock alerts |
| **My Audits** | Manage assigned audit cases | Table view with Case #, Taxpayer, Type, Status, Due Date, Actions (Open, Update, Submit); Filters by status, type, date range |
| **Entry Conference Management** | Schedule, conduct, and record initial meetings | Calendar widget, scheduling form, meeting minutes editor, audio upload, document checklist, submit for approval |
| **Audit Execution Workspace** | Conduct desk/comprehensive/TP audits | Split view: Case file (left), Working papers (right); Data analytics tools; Document upload; Query management |
| **Exit Conference Management** | Schedule and conduct final meetings | Similar to Entry Conference but with draft report attachment and finalization workflow |
| **Draft Reports** | Prepare and submit audit reports | WYSIWYG editor, template selector, approval workflow tracker, version history |
| **Taxpayer Communication** | Send/receive messages and documents | Inbox, secure messaging, document exchange, notification log |

### 3.2 Team Leader Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **Team Dashboard** | Monitor team performance | Team workload distribution, case status summary, approval queues, productivity metrics |
| **Approvals** | Review and approve submissions | List of pending approvals (Entry/Exit Conferences, Draft Reports, QA Reports); Approve/Reject with comments |
| **Team Audits** | View all team cases | Similar to Auditor view but across all team members, with oversight flags |

### 3.3 Process Owner Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **Process Dashboard** | Overview of audit lifecycle | Case selection funnel, capacity vs. workload, pending objections, notice tracking |
| **Case Selection** | Select and prioritize audit cases | Combined list from risk engine, internal/external requests; Priority ranking; Capacity calculator; Treatment plan assignment |
| **Objection Handling** | Validate taxpayer objections | Objection details, evidence review, Validity Check workflow (Accept/Reject/Fraud), account adjustment |
| **Notice Management** | Generate and track notices | Template selector, dynamic preview, unique ID generation, batch printing, delivery tracking |

### 3.4 Director / Authorized Official Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **Executive Dashboard** | High-level oversight | Audit completion metrics, assessment values, objection trends, fraud cases |
| **Final Approvals** | Approve final reports and assessments | Approval queue with monetary thresholds, document review, electronic signature, audit trail |
| **Assessment Issuance** | Issue official assessment notices | Final assessment preview, principal/penalty/interest calculation, send to taxpayer |
| **Account Adjustment** | Adjust taxpayer accounts | Taxpayer liability update, principal/penalty/interest adjustment, audit trail logging |

### 3.5 Joint Audit Team Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **Joint Audit Workspace** | Collaborate across agencies | Shared case file, participating authorities list, real-time activity feed |
| **Shared Document Repository** | Manage shared documents | Version control (check-in/check-out), conflict alerts, document categorization, search |
| **CAAT Dashboard** | Run computer-assisted audit | Eligibility check, automated analysis, anomaly detection, ratio comparisons |
| **Consolidated Reporting** | Generate multi-zone reports | Consolidated view, segregated per-zone reports, aggregate tax calculation |

### 3.6 Taxpayer / Tax Agent Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **Taxpayer Dashboard** | View audit status | Active audits, notifications, pending requests, upload history |
| **Meeting Confirmations** | Confirm or reschedule meetings | Meeting details, Confirm/Reschedule buttons, calendar integration |
| **Document Upload** | Submit requested documents | Upload zone, file validation, submission history, status tracking |
| **Objection Submission** | Raise objections to assessments | Objection form, evidence upload, submission confirmation, status tracking |
| **Communication Hub** | Secure messaging with auditor | Inbox, message composer, attachment support, read receipts |

### 3.7 QA Team Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **QA Dashboard** | Manage QA reviews | Sampling configuration, selected cases, QA team assignment, follow-up tracking |
| **QA Review Workspace** | Review completed audits | Side-by-side comparison with original audit, rating form, findings documentation |
| **QA Report Generation** | Document QA findings | Draft report, approval workflow, exit conference scheduling (QA) |

### 3.8 Admin Staff Pages

| **Page Name** | **Purpose** | **Key Elements** |
| :--- | :--- | :--- |
| **Notice Printing** | Manage physical notice delivery | Batch selection, print queue, delivery confirmation |
| **Undelivered Tracking** | Handle undelivered notices | Undelivered list, alternative outreach (post, affix, newspaper), escalation |
| **Bulk Operations** | Manage mass actions | Bulk notice generation, bulk case assignment, bulk document printing |

---

## 4. Design Principles & Best Practices

### 4.1 Visual Design Principles

| **Principle** | **Application** |
| :--- | :--- |
| **Clarity** | Use clear hierarchy (headings, subheadings, body text). Ensure every page has a single primary action (CTA). |
| **Consistency** | Maintain consistent colors, typography, spacing, and component behavior across all pages. |
| **Professionalism** | Use a clean, modern design with generous white space. Avoid clutter. Colors should be muted and trustworthy (blues, grays, whites). |
| **Accessibility** | WCAG 2.1 AA compliance. High contrast, keyboard navigation, screen reader support, responsive design. |
| **Efficiency** | Minimize clicks. Design for power users. Provide bulk actions, keyboard shortcuts where possible. |
| **Feedback** | Every action should have immediate visual feedback (loading states, success/error messages, progress indicators). |
| **Security** | Role-based access control (RBAC) enforced at the UI level. Sensitive actions require confirmation. |

### 4.2 Color Palette

| **Color** | **Hex Code** | **Usage** |
| :--- | :--- | :--- |
| **Primary Navy** | `#0F172A` | Sidebar, headers, primary text |
| **Primary Blue** | `#2563EB` | Primary CTAs, active states, links |
| **Blue Light** | `#EFF6FF` | Backgrounds, hover states, selected items |
| **Success Green** | `#10B981` | Positive statuses, approval, confirmation |
| **Warning Yellow** | `#F59E0B` | Pending statuses, warnings, awaiting action |
| **Danger Red** | `#EF4444` | Overdue, errors, urgent actions, rejection |
| **Neutral Gray** | `#64748B` | Secondary text, borders, disabled states |
| **Background** | `#F1F5F9` | Page backgrounds |
| **White** | `#FFFFFF` | Card backgrounds, form fields |

### 4.3 Typography

| **Element** | **Font** | **Size** | **Weight** |
| :--- | :--- | :--- | :--- |
| **Headings (H1)** | Inter / Segoe UI | 28-32px | 700 |
| **Headings (H2)** | Inter / Segoe UI | 20-24px | 600 |
| **Headings (H3)** | Inter / Segoe UI | 16-18px | 600 |
| **Body Text** | Inter / Segoe UI | 14px | 400 |
| **Small Text** | Inter / Segoe UI | 12px | 400 |
| **Labels** | Inter / Segoe UI | 14px | 500 |
| **CTA Buttons** | Inter / Segoe UI | 14px | 600 |

### 4.4 Component Standards

| **Component** | **Design Standard** |
| :--- | :--- |
| **Buttons** | Pill-shaped (`border-radius: 10px`); Primary = solid blue; Secondary = outline; Danger = solid red; Disabled = gray. |
| **Cards** | White background, `border-radius: 16px`, `box-shadow: 0 1px 3px rgba(0,0,0,0.04)`, hover shadow for interactivity. |
| **Forms** | Clean labels above fields; `border-radius: 10px`; `padding: 8px 14px`; focus state with blue ring. |
| **Tables** | Clean, striped rows; header background `#F8FAFC`; hover row highlight; status pills for statuses. |
| **Status Pills** | Colored dots + text; background matches status (e.g., `bg-green-light` for completed). |
| **Modals** | Centered, overlay background, `border-radius: 16px`, clear header/body/footer sections. |
| **Sidebar** | Fixed width (260px), dark background, nested navigation with icons, user profile at bottom. |
| **Notifications** | Badge icons on bell, separate notification center with read/unread status. |
| **Alerts** | Colored banners at top of page (🔴 Critical, 🟡 Warning, 🟢 Success, 🔵 Info). |

---

## 5. Key UX Flows

### 5.1 Entry Conference Flow
```
Dashboard → Entry Conference Module → Schedule (Page) → Send Invitation → Taxpayer Confirms → Conduct Meeting (Page) → Submit to Team Leader → Team Leader Approves (Page) → Send Documentation → Taxpayer Confirms Receipt
```

### 5.2 Full Audit Lifecycle (Visual)
```
[Annual Plan] → [Case Selection] → [Assignment] → [Entry Conference] → [Audit Execution] → [Exit Conference] → [Report Finalization] → [Notice Issuance] → [Assessment] → [Objection Handling] → [Closure] → [QA Review]
```

---

## 6. Professional Industry Requirements

### 6.1 Tax Administration Specifics
- **Statute Clocks**: Display remaining days prominently on case pages. Color-coded urgency (🔴 Overdue, 🟡 Approaching, 🟢 On Track).
- **Audit Trail**: Every action (view, edit, approve, send) logged with timestamp and user ID. Visible to authorized users.
- **Separation of Duties (SoD)**: System must enforce that the same user cannot prepare and approve a document. UI should reflect this (e.g., approver cannot edit, preparer cannot approve).
- **Digital Signatures**: Support electronic signature for taxpayer confirmations and official approvals.
- **Multi-Zone Support**: For taxpayers operating across regions, UI must support consolidated view and per-zone segregation.

### 6.2 Security & Compliance
- **Role-Based Access Control (RBAC)**: UI elements should dynamically render based on user role (e.g., Admin links only visible to Admins).
- **Session Management**: Auto-logout after inactivity (configurable, default 30 min). Session timer visible.
- **Encryption**: All sensitive data (taxpayer information, financial records) encrypted in transit (HTTPS) and at rest.
- **Audit Logging**: All access and actions logged for compliance and forensic review.

### 6.3 Accessibility
- **WCAG 2.1 AA Compliance**: Color contrast ratios (minimum 4.5:1 for normal text), keyboard navigable, screen reader compatible, focus indicators visible.
- **Responsive Design**: Support desktop (primary), tablet, and mobile views.
- **Multilingual Support**: Design should support RTL languages and future translation capabilities.

---

## 7. Page Layout Template

### 7.1 Standard Page Structure
```
+================================================================================+
|  SIDEBAR (260px)          |  MAIN CONTENT AREA                                 |
|  ┌──────────────────────┐ |  ┌──────────────────────────────────────────────────┐ |
|  │ Brand Logo           │ |  │  TOP HEADER                                     │ |
|  │ Navigation Items     │ |  │  [Page Title]              [Notifications] [User]│ |
|  │  - Dashboard         │ |  ├──────────────────────────────────────────────────┤ |
|  │  - My Audits         │ |  │  STATUTE CLOCK / ALERT BANNER (if applicable)    │ |
|  │  - Entry Conferences │ |  ├──────────────────────────────────────────────────┤ |
|  │  - Exit Conferences  │ |  │  KPI CARDS (if dashboard)                       │ |
|  │  - Reports           │ |  ├──────────────────────────────────────────────────┤ |
|  │                      │ |  │  FILTER BAR (if list/table page)               │ |
|  │  ────────────────    │ |  ├──────────────────────────────────────────────────┤ |
|  │  User Profile        │ |  │  MAIN CONTENT (Cards, Tables, Forms, Workspace)  │ |
|  │  [Avatar] Name       │ |  │                                                   │ |
|  │  Role                │ |  │                                                   │ |
|  └──────────────────────┘ |  └──────────────────────────────────────────────────┘ |
+================================================================================+
```

---

## 8. UI Design Deliverables

### 8.1 Required Deliverables
1. **High-Fidelity Mockups** for each page in the inventory (Figma, Sketch, or Adobe XD)
2. **Interactive Prototype** showing key user flows (Entry Conference, Objection Handling, Notice Issuance)
3. **Design System / Component Library** documenting colors, typography, spacing, and components
4. **Responsive Designs** for desktop, tablet, and mobile views
5. **Accessibility Audit** report confirming WCAG 2.1 AA compliance
6. **User Journey Maps** for each role (Auditor, Team Leader, Process Owner, Director, Taxpayer, QA Team)

### 8.2 Design Review Checklist
- [ ] All pages follow the standard layout template
- [ ] Sidebar navigation correctly reflects user role (Admin hidden for non-Admins)
- [ ] Status indicators (pills) are consistent across all pages
- [ ] CTAs are clear and prominently placed
- [ ] Forms have validation and clear error messages
- [ ] Responsive breakpoints are handled (1024px, 768px, 480px)
- [ ] Font sizes meet accessibility standards
- [ ] Color contrasts meet WCAG 2.1 AA standards
- [ ] All icons are meaningful and consistent
- [ ] Loading states and empty states are designed

---

## 9. Additional Considerations

### 9.1 UX Best Practices
- **Progressive Disclosure**: Show only what's needed; hide advanced options behind "Show More" or "Advanced" toggles.
- **Contextual Help**: Tooltips or help icons for complex fields (e.g., "What is a treatment plan?").
- **Bulk Operations**: Support batch actions (e.g., bulk notice printing, bulk assignment) with confirmation dialogs.
- **Notifications**: In-app notifications for all pending actions, deadlines, and approvals. Optional email/SMS integration.
- **Search**: Global search for cases, taxpayers, and documents. Advanced filters for specific modules.

### 9.2 Technical Considerations
- **Performance**: Pages should load within 2 seconds. Large tables should support pagination and virtual scrolling.
- **API Integration**: Design should support RESTful API integration with clear loading states and error handling.
- **Real-Time Updates**: For joint audit workspaces, support real-time updates (WebSockets or polling with notifications).
- **Print-Friendly**: Notice and report pages should have print stylesheets for official documentation.

---

## 10. Final Design Principles

| **Principle** | **Description** |
| :--- | :--- |
| **Trustworthy** | Professional, clean design that builds confidence in the tax authority. |
| **Efficient** | Power users (auditors) should complete tasks with minimal clicks. |
| **Accurate** | Clear data display to prevent errors in tax calculations and assessments. |
| **Secure** | Role-based access and audit trails enforced at the UI level. |
| **User-Friendly** | Intuitive navigation with clear CTAs and helpful feedback. |
| **Accessible** | Inclusive design for all users, regardless of ability. |
| **Responsive** | Adaptable to different screen sizes and devices. |

---

*This prompt serves as the complete UI/UX specification for the Tax Audit Management System. The designer should use this as the single source of truth for all design decisions.*
