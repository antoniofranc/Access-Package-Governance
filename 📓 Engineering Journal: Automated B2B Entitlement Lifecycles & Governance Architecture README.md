# 📓 Engineering Journal: Automated B2B Entitlement Lifecycles & Governance Architecture
 
**Author:** Identity & Access Management (IAM) Security Engineer  
**Date:** Day 4 Lab Deployment  
**Environment:** Microsoft Entra ID Hybrid-Cloud Sandbox (P2 Tier)  
**Core Frameworks:** Entra ID Entitlement Management, B2B External Identity Lifecycles, NIST Least Privilege Compliance
 
---
 
## Table of Contents
 
1. [Entry 1 — Scoping & Resource Provisioning](#-entry-1-scoping--resource-provisioning)
   - [Step 1 — Building the Target Resource Perimeter](#️-step-1-building-the-target-resource-perimeter)
   - [Step 2 — Isolating the Governance Vault (External Collaboration Catalog)](#️-step-2-isolating-the-governance-vault-external-collaboration-catalog)
2. [Entry 2 — Engineering the Contractor Access Package](#-entry-2-engineering-the-contractor-access-package)
   - [Step 1 — Programming the External Request Policy](#️-step-1-programming-the-external-request-policy)
   - [Step 2 — Enforcing Lifecycle & Access Reviews](#-step-2-enforcing-lifecycle--access-reviews)
3. [Entry 3 — Advanced Incident & Workaround Log](#-entry-3-advanced-incident--workaround-log)
4. [Entry 4 — Executing the Full Lifecycle Validation Loop](#-entry-4-executing-the-full-lifecycle-validation-loop)
   - [Step 1 — Sponsor Certification & Sign-Off](#️-step-1-sponsor-certification--sign-off)
   - [Step 2 — Simulating Ongoing Governance (Access Reviews)](#-step-2-simulating-ongoing-governance-access-reviews)
   - [Step 3 — Simulating Expiry, Leaver Clean-Up & Hygiene Policy](#-step-3-simulating-expiry-leaver-clean-up--hygiene-policy)
5. [Project Summary & Architectural Sign-Off](#-project-summary--architectural-sign-off)
---
 
## 📅 Entry 1: Scoping & Resource Provisioning
 
### 🎯 Objective
 
Architect a secure, automated boundary around a cross-functional internal project environment named **Project Alpha**. The system must automate:
 
| Lifecycle Phase | Description |
|---|---|
| **Joiner** | External vendor onboarding |
| **Mover** | Continuous access recertification |
| **Leaver** | Strict automated de-provisioning and guest account cleanup |
 
**Constraint:** Zero IT administrator intervention at any phase.
 
---
 
### 🛠️ Step 1: Building the Target Resource Perimeter
 
Before configuring any identity logic, I provisioned an isolated corporate data boundary to act as the target resource layer.
 
| Resource | Type | Purpose |
|---|---|---|
| `Project-Alpha-Site` | SharePoint Team Site | Secure project file storage |
| `Project Alpha` | Microsoft 365 Group / Teams | Dedicated project collaboration channel |
| `Project-Tracker-App` | Non-gallery enterprise application | Simulates a proprietary internal project management app |
 
> **Security protocol:** All resources were initialized with **0 members** and assigned only to the primary Global Administrator account — ensuring clean, dependency-free automation testing downstream.
 
---
 
### 🗂️ Step 2: Isolating the Governance Vault (External Collaboration Catalog)
 
In Entra ID, catalogs act as **secure silos for delegated asset administration**.
 
1. Navigate to **Identity Governance → Entitlement Management → Catalogs → New Catalog**.
2. Name the catalog: `External Collaboration`.
3. Toggle **Enabled for external users = Yes** — this allows cross-tenant external contractors to interact with the asset directory.
4. Load the three Project Alpha resources into the catalog with least-privileged role bindings:
| Resource | Role Assigned | Access Level |
|---|---|---|
| `Project-Alpha-Site` | Members | Maps to Contribute — document upload access |
| `Project Alpha` Group | Member | Standard team collaboration |
| `Project-Tracker-App` | User / `msiam_access` | Baseline application runtime authorization |
 
---
 
## 📅 Entry 2: Engineering the Contractor Access Package
 
I bundled the catalog inventory into a single request token named **`Contractor Access – Project Engagement`** — allowing contractors to request their entire suite of project tools in one action.
 
---
 
### 🛡️ Step 1: Programming the External Request Policy
 
**Who can request access:**  
Set to `For users not in your directory → Any organization` — exposes a secure, public-facing registration endpoint for external vendors.
 
**Approval gates:**
 
| Setting | Configuration |
|---|---|
| Approval stages | 1-stage |
| Primary approver | Business sponsor (`Terry.Manager`) |
| Decision window | 3 business days |
| Escalation rule | If sponsor does not act within 2 days → auto-escalates to fallback Global Administrator |
 
**Requestor information auditing:**  
Injected a mandatory compliance question into the request form:
 
> *"Please provide the contractor's project name, engagement start date, and expected end date."*
 
This logs required compliance documentation into the directory ledger before a request can be submitted.
 
---
 
### ⏳ Step 2: Enforcing Lifecycle & Access Reviews
 
**Assignment expiration:**  
Hard expiration set to **180 days** — eliminates permanent access creep with no option to remain assigned indefinitely.
 
**Continuous auditing:**  
Configured an automated **quarterly access review cycle** with the following settings:
 
| Setting | Value |
|---|---|
| Reviewer | Business sponsor (`Terry.Manager`) |
| Frequency | Quarterly (every 90 days) |
| No-response action | **Remove access** (default-deny on inaction) |
| Behavior | If the sponsor fails to actively recertify during the audit window, the Entra engine automatically terminates the access package |
 
---
 
## 📅 Entry 3: Advanced Incident & Workaround Log
 
### 🚨 The Incident: Direct Administrative Assignment Lockout
 
While attempting to validate the approval engine, I navigated to the Access Package dashboard, opened the **Assignments** tab, and clicked **+ New Assignment** to force-inject a test email (`contractor-test@gmail.com`) as an administrator. The portal repeatedly failed and blocked execution.
 
---
 
### 🔍 Root-Cause Diagnosis
 
A Microsoft security restriction blocks administrators from using the **Direct Assignment** button to manually insert a completely unprovisioned external identity into a policy that requires active manager approval. This design constraint exists for two reasons:
 
- Prevents IT from accidentally **bypassing the business approval chain**
- Prevents generation of **corrupt guest state objects** in the directory
---
 
### 🛠️ Resolution & Architectural Pivot
 
To bypass this limitation and validate the B2B guest account lifecycle properly, I replicated a **true external contractor endpoint experience**:
 
1. Extracted the unique **public-facing My Access portal link** directly from the package overview page.
2. Launched an isolated **InPrivate browser window** to isolate credentials.
3. Authenticated as an external identity using a personal email address via the **One-Time Passcode (OTP)** federated authentication stream.
4. Completed the request form wizard — provided the required business justification and compliance answers.
**Result:**
 
```
Self-service workflow executed successfully
    → Admin console block bypassed via the correct external request path
    → Guest identity metadata wrapper provisioned
    → Request locked into verified 'Pending Approval' state in the queue
```
 
---
 
## 📅 Entry 4: Executing the Full Lifecycle Validation Loop
 
### ✍️ Step 1: Sponsor Certification & Sign-Off
 
1. Authenticated into `myapps.microsoft.com` as the business sponsor, `Terry.Manager`.
2. Opened the **Approvals** pane. Expanded the contractor's data card and audited their form answers.
3. Confirmed business justification alignment, submitted a compliance logging note, and clicked **Submit**.
4. Returned to the Global Admin console and checked the assignments panel.
**Telemetry result — fully automated pipeline, zero manual IT intervention:**
 
```
Sponsor approval submitted
    → New external guest user object created
         (format: yourname_domain.com#EXT#@<tenant>.onmicrosoft.com)
    → Profile synchronized into Project Alpha group
    → SharePoint site access activated
    → Assignment status: ✅ Delivered
```
 
---
 
### 🧪 Step 2: Simulating Ongoing Governance (Access Reviews)
 
1. While logged in as `Terry.Manager`, navigated to the **Access Reviews** hub.
2. Located the automated quarterly audit card for the `Contractor Access – Project Engagement` package.
3. Selected the contractor's profile. Clicked **Approve** to verify ongoing contract status.
4. Submitted review — directory compliance logs updated.
> **Screenshot:** Access review card for the contractor package — sponsor certification submitted with approval note
 
---
 
### 🧹 Step 3: Simulating Expiry, Leaver Clean-Up & Hygiene Policy
 
To verify the automatic **Leaver** safety mechanisms, I forced an immediate termination by moving the contractor's assignment end date to a past timestamp.
 
The Entra ID engine executed the cleanup architecture in near-real-time:
 
| Phase | Action Observed |
|---|---|
| **Perimeter lockdown** | Assignment status → `Expired`. `Block sign-in` parameter automatically toggled to `Yes`. |
| **Resource purge** | SharePoint site returned `HTTP 403 Access Denied`. Contractor account removed from the Microsoft Teams channel. |
| **Audit log ingestion** | Three system log entries recorded: `Assignment expired`, `Resource role removed`, `User blocked from sign-in`. |
| **Long-term deletion** | Guest lifecycle policy confirmed: if the account remains soft-blocked with no new assignments for **30 days**, the cleanup engine triggers a hard-delete — permanently purging the identity from the graph database. |
 
> **Screenshot:** Entra ID → Identity Governance → Audit logs — filtered by `Category = EntitlementManagement`, showing all three cleanup events timestamped in sequence
 
**Verified under:** Entra ID → **External Collaboration Settings → Guest user lifecycle management**.
 
---
 
## 🏁 Project Summary & Architectural Sign-Off
 
By completing Day 4, I successfully engineered an advanced, end-to-end **B2B Identity Governance lifecycle** covering all three phases of the Joiner-Mover-Leaver model.
 
| Capability Demonstrated | Outcome |
|---|---|
| Resource perimeter provisioning | SharePoint, Teams, and non-gallery app loaded into isolated governance catalog |
| External access package build | `Contractor Access – Project Engagement` configured with least-privilege roles |
| External request policy with approval gates | 3-day decision window, sponsor approval, 2-day escalation to fallback admin |
| Lifecycle enforcement | 180-day hard expiry, quarterly access review, default-deny on no response |
| Incident diagnosis & workaround | Direct assignment lockout root-caused; OTP self-service path used to validate B2B flow |
| Full lifecycle validation | Joiner → approval → Delivered → access review → Leaver → automated cleanup confirmed |
| Audit trail verification | All three cleanup log entries confirmed in Entra ID audit log |
| Guest account hygiene policy | 30-day soft-block → hard-delete confirmed in External Collaboration settings |
 
This architecture eliminates manual onboarding tickets, delegates authorization directly to business leads, prevents privilege creep through access reviews, and enforces an **automated, self-cleaning Zero Trust directory**.
 
---
 
*Engineering Journal — Microsoft Entra ID IAM Lab | Entitlement Management | Day 4 Portfolio Artifact*
