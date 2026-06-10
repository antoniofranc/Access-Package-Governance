# Access Package Governance
 
**Contractor Role — Design, Build & Test Documentation**  
Final IAM Project — Day 4 Portfolio Artifact
 
> Microsoft Entra ID Governance &nbsp;|&nbsp; Entitlement Management  
> May 2026
 
---
 
## Table of Contents
 
1. [Overview and Design Goals](#1-overview-and-design-goals)
   - [1.1 Design Principles](#11-design-principles)
2. [Access Package Design](#2-access-package-design)
   - [2.1 Package Configuration](#21-package-configuration)
   - [2.2 Resource Roles — Detail](#22-resource-roles--detail)
   - [2.3 Request Policy — Who Can Request](#23-request-policy--who-can-request)
   - [2.4 Approval Settings](#24-approval-settings)
   - [2.5 Lifecycle Settings](#25-lifecycle-settings)
3. [Guest Account Behavior](#3-guest-account-behavior)
4. [Conditional Access Controls for Contractors](#4-conditional-access-controls-for-contractors)
5. [Lab Build Steps](#5-lab-build-steps)
   - [Step 1 — Verify the Catalog](#step-1--verify-the-catalog-has-the-required-resources)
   - [Step 2 — Create the Access Package](#step-2--create-the-access-package)
   - [Step 3 — Test the Request Flow](#step-3--test-the-request-flow)
   - [Step 4 — Approve the Request](#step-4--approve-the-request-as-sponsor)
   - [Step 5 — Verify Guest Account Creation](#step-5--verify-guest-account-creation)
   - [Step 6 — Test Contractor Sign-In](#step-6--test-contractor-sign-in)
   - [Step 7 — Simulate Access Review](#step-7--simulate-access-review)
   - [Step 8 — Simulate Expiry and Offboarding](#step-8--simulate-expiry-and-offboarding)
6. [Full Governance Workflow — Narrative](#6-full-governance-workflow--narrative)
   - [6.1 Engagement Start](#61-engagement-start)
   - [6.2 Provisioning](#62-provisioning)
   - [6.3 Active Engagement](#63-active-engagement)
   - [6.4 Engagement End](#64-engagement-end)
7. [Design Decisions and Rationale](#7-design-decisions-and-rationale)
---

## 1. Overview and Design Goals
 
This document covers the design, build, and test of an access package for the **Contractor role** in Microsoft Entra ID Governance — Entitlement Management. It serves as a portfolio artifact demonstrating real-world identity governance competency.
 
Contractors present a distinct governance challenge compared to employees:
 
- They are **external identities** with no standing directory presence
- They work on **time-limited engagements**
- They should have access to **only specific project resources** — not broad organizational resources
- Their **departure must be fully automated** with no manual clean-up steps required
The access package solves all of these challenges in a single governance construct: it provisions a scoped guest account, assigns exactly the right resource roles, enforces access controls via Conditional Access, requires periodic human certification, and automatically removes all access and cleans up the guest account when the engagement ends.
 
---
 
### 1.1 Design Principles
 
| Principle | Description |
|---|---|
| **Least privilege** | Contractors receive only the resources required for their specific project — not broad organizational access |
| **Time-bound by default** | All access expires at 180 days with no option to remain indefinitely; renewal requires explicit sponsor action |
| **Zero standing access** | No permanent resource role assignments; all access flows through the access package lifecycle |
| **Automated cleanup** | Guest account blocking and deletion happen automatically on expiry — no manual admin steps required |
| **Auditability** | Every request, approval, assignment, review, and removal is logged in the Entra ID audit log |
 
---
 
## 2. Access Package Design
 
### 2.1 Package Configuration
 
| Setting | Value |
|---|---|
| **Package name** | `Contractor Access — Project Engagement` |
| **Catalog** | External Collaboration *(or General if no dedicated catalog)* |
| **Description** | Grants project-scoped access to contractors for the duration of their engagement. Managed by the business sponsor. Expires after 180 days. |
| **Hidden** | No — visible in the My Access portal for managers to request on behalf of contractors |
| **Resource 1** | SharePoint: Project site — Contribute role |
| **Resource 2** | Teams: Project channel — Member role |
| **Resource 3** | Line-of-business application — Contributor role *(e.g. project tracking app)* |
 
---
 
### 2.2 Resource Roles — Detail
 
Each resource is assigned at the **minimum permission level** required for the contractor to perform their work.
 
| Resource | Role Assigned | Rationale |
|---|---|---|
| SharePoint project site | `Contribute` | Contractors need to read documents and upload deliverables. Owner or Full Control would allow site settings changes — not needed. |
| Microsoft Teams channel | `Member` | Contractors need to participate in project communication. Owner role would allow channel management — not appropriate. |
| Line-of-business app | `Contributor` | Contractors need to create and edit project records. Admin or Owner would expose configuration and other projects — not appropriate. |
 
---
 
### 2.3 Request Policy — Who Can Request
 
| Setting | Value |
|---|---|
| **Who can request** | For users not in your directory *(external contractors are not yet in the tenant)* |
| **Scope** | All connected organizations — or a specific connected organization if the contractor's employer is a known partner |
| **Who initiates** | Manager / business sponsor requests on behalf of the contractor |
| **Self-service** | Disabled — contractors cannot request their own access; a manager must initiate |
| **Requestor justification** | Required — manager must state the project name, engagement dates, and scope of work |
 
---
 
### 2.4 Approval Settings
 
| Setting | Value |
|---|---|
| **Approval required** | Yes |
| **Stages** | Single-stage |
| **First approver** | Business sponsor *(specific named approver — not manager, since external users have no Entra ID manager attribute)* |
| **Fallback approver** | Identity Governance Administrator — if sponsor does not respond within 3 days |
| **Decision window** | 3 business days |
| **Sponsor justification required** | Yes — sponsor must confirm scope and dates |
| **Alternate approvers** | Yes — forwarded at day 2 *(half-life of the 3-day window)* |
 
---
 
### 2.5 Lifecycle Settings
 
| Setting | Value |
|---|---|
| **Access expires** | Number of days: `180` |
| **Allow extension** | Yes — sponsor must approve; max extension to original end date |
| **Extension approval** | Yes — same approval workflow as initial request |
| **User can request specific timeline** | Yes — manager can set a shorter end date if engagement is less than 180 days |
| **Access review** | Yes — quarterly (every 90 days) |
| **Review type** | Self-review by sponsor (sponsor certifies each contractor is still engaged) |
| **No response action** | Remove access *(default deny on inaction — safer for external identities)* |
| **Email notifications** | Enabled — reminder at 14 days before expiry and 1 day before expiry |
 
---
 
## 3. Guest Account Behavior
 
When a contractor's request is approved, Entra ID Governance automatically creates a **B2B guest account** for the contractor's external identity if one does not already exist.
 
| Behavior | Detail |
|---|---|
| **Account creation** | Guest account created automatically when the assignment is approved. The contractor receives an access delivery email with a link to the My Access portal. |
| **UPN format** | `contractor@externalcompany.com#EXT#@yourtenant.onmicrosoft.com` |
| **MFA on first sign-in** | The baseline CA policy `Require MFA for all users` intercepts the first sign-in and requires MFA registration via Microsoft Authenticator. |
| **Session persistence** | No persistent browser session — the contractor must re-authenticate periodically. |
| **Directory visibility** | Guest accounts appear in **Entra ID → Users**, tagged as `Guest` user type and filterable separately from member accounts. |
| **Multiple packages** | If the same contractor is later assigned to a second project, the existing guest account is reused — no duplicate created. |
| **Account blocking** | When the last access package assignment expires or is removed, Entra ID Governance automatically blocks sign-in for the guest account. |
| **Account deletion** | 30 days after blocking, the guest account is permanently deleted. Configurable in **Entra ID Governance settings → External users**. |

> **Screenshot:** Entra ID → Users — filtered by `User type = Guest`, showing contractor account with `#EXT#` UPN format

 
---
 
## 4. Conditional Access Controls for Contractors
 
Contractor guest accounts are subject to the same **baseline Conditional Access policies** as all users in the tenant. The following policies from Day 2 are particularly impactful for external identities.
 
| Policy | Impact on Contractors |
|---|---|
| **Require MFA for all users** | Contractors must register MFA on first sign-in. Without an existing Entra ID account, they use Microsoft Authenticator via the Microsoft identity platform. This is the most important control for external identities. |
| **Block legacy authentication** | Prevents contractors from using basic auth protocols. All contractor access must go through modern authentication — enforced automatically. |
| **No persistent browser session** | Contractors cannot check "Stay signed in". Each new browser session requires re-authentication — limits the impact of an unattended contractor workstation. |
| **Compliant device or MFA fallback** | Contractors on unmanaged personal devices will always fall back to MFA (they cannot satisfy device compliance without Intune enrollment). This is expected and acceptable. |
| **Require MFA for Azure management** | Prevents contractors from accessing the Azure portal without MFA, even if they somehow have Azure RBAC assignments. |
 
> **Consideration:** For highly sensitive contractor engagements, a contractor-specific CA policy can be created targeting the contractor security group with stricter controls such as: block access from non-corporate countries, require token protection (sign-in session bound to device), or restrict access to specific named applications only.
 
---
 
## 5. Lab Build Steps
 
The following steps walk through building the contractor access package in a Microsoft Entra ID lab tenant. Screenshot placeholders mark where evidence should be captured for the portfolio.
 
---
 
### Step 1 — Verify the Catalog Has the Required Resources
 
1. Navigate to **Identity Governance → Entitlement Management → Catalogs**.
2. Open the target catalog — or create one named `External Collaboration` if it does not exist:
   - **New catalog** → name, description, `Enable for external users = Yes`
3. In the catalog, select **Resources → Add resources**.
4. Add: the SharePoint project site, the Teams team, and the line-of-business application.
5. Verify all three appear in the catalog Resources list before proceeding.

> **Screenshot:** Catalog resources list showing SharePoint site, Teams team, and application added
 
---

### Step 2 — Create the Access Package
 
1. Navigate to **Identity Governance → Entitlement Management → Access packages → New access package**.
2. **Basics tab:** Name = `Contractor Access — Project Engagement`. Select the External Collaboration catalog. Add description. Select **Next**.
3. **Resource roles tab:** Add each resource from the catalog — select the minimum role for each (Contribute / Member / Contributor). Select **Next**.
4. **Requests tab:** Who can get access = `For users not in your directory`. Scope = All connected organizations. Enable approval. Set approval to 1 stage, sponsor as approver. Decision window = 3 days. Enable requestor justification. Select **Next**.
5. **Requestor information tab:** Add a required question — *"Please provide the contractor's project name, engagement start date, and expected end date."* Select **Next**.
6. **Lifecycle tab:** Access expires = `180 days`. Allow extension = Yes. Enable access review, frequency = quarterly, reviewer = sponsor. No response action = `Remove access`. Select **Next**.
7. **Review + create:** Review all settings. Select **Create**.
> **Screenshot:** Access package overview page showing all three resource roles and both policies (request and review)
 
---
 
### Step 3 — Test the Request Flow
 
1. Navigate to the access package. Select **Assignments → New assignment**.
2. Select **Policy:** the request policy created in Step 2.
3. Under **Users:** enter the external contractor's email address *(external identity not yet in the directory)*.
4. Set an end date shorter than 180 days for testing (e.g. 14 days).
5. Add justification: `Test contractor engagement — project alpha`.
6. Select **Add**. The assignment is created in a `Pending approval` state.
> **Screenshot:** Assignment creation screen showing pending approval state for external contractor email
 
---
 
### Step 4 — Approve the Request (as Sponsor)
 
1. Open a new **InPrivate/Incognito** window. Sign in as the designated sponsor account.
2. Navigate to `myaccess.microsoft.com`. Select **Approvals** from the left navigation.
3. Find the pending contractor request. Review the justification.
4. Select **Approve**. Enter a justification: `Verified contractor engagement for project alpha. Approved for 14-day test period.`
5. Return to the admin account. Navigate to the access package **Assignments** tab. Confirm the assignment status is now `Delivered`.
> **Screenshot:** My Access portal — approvals page showing approved contractor request with justification  
> **Screenshot:** Access package assignments tab — contractor assignment showing `Delivered` status with start and end dates
 
---
 
### Step 5 — Verify Guest Account Creation
 
1. Navigate to **Entra ID → Users → All users**.
2. Filter by `User type = Guest`. Locate the contractor guest account.
3. Open the account. Verify:
   - Display name and UPN (`#EXT#` format)
   - Assigned licenses: **none** — contractors should not consume licenses
   - Group memberships: **none directly** — access flows through the package only
4. Under the user, navigate to **Access package assignments** — confirm the package assignment appears.
> **Screenshot:** Guest user profile showing UPN format, `User type = Guest`, and access package assignment
 
---
 
### Step 6 — Test Contractor Sign-In
 
1. Open an **InPrivate/Incognito** window. Navigate to `myapps.microsoft.com`.
2. Sign in as the contractor using the external identity's credentials.
3. Verify: MFA registration prompt appears on first sign-in. Complete MFA setup with Microsoft Authenticator.
4. After MFA, the contractor should see the apps assigned via the access package in My Apps.
5. Verify access to the SharePoint site. Verify access to the Teams channel.
> **Screenshot:** Contractor's My Apps portal showing SharePoint and app tiles from the access package  
> **Screenshot:** Sign-in logs for the contractor guest account — Conditional Access tab showing MFA required and satisfied
 
---
 
### Step 7 — Simulate Access Review
 
1. Navigate to **Identity Governance → Access reviews** — find the review created for this package.
2. Select the review. Under **Reviewers**, select the sponsor account.
3. Open the review as the sponsor (via the review notification email link, or from `myaccess.microsoft.com → Access reviews`).
4. For each contractor listed, select **Approve** (contractor is still engaged) or **Deny** (contractor is no longer engaged). Add a review comment. Submit.
5. Back in the admin view, confirm the review result is applied: approved = access continues; denied = access removed.
> **Screenshot:** Access review showing contractor listed for certification by sponsor
 
---
 
### Step 8 — Simulate Expiry and Offboarding
 
1. Manually expire the test assignment: open the assignment, change the end date to today or a past date. *(Or wait for the 14-day test period to naturally expire.)*
2. Navigate to **Assignments** — confirm the assignment status changes to `Expired`.
3. Navigate to **Entra ID → Users** — find the contractor guest account. Confirm `Block sign-in = Yes`.
4. Check the audit log: **Identity Governance → Audit logs** — filter by `Category = EntitlementManagement`. Confirm entries for: assignment expiry, resource removal, and guest account blocking.
> **Screenshot:** Audit log entries showing assignment expired, resources removed, and guest account blocked
 
---
 
## 6. Full Governance Workflow — Narrative
 
This section describes the contractor access package lifecycle as a complete narrative, suitable for explaining the governance model to an IAM lead, auditor, or new team member.
 
---
 
### 6.1 Engagement Start
 
A project manager identifies a need for an external contractor. The manager navigates to the **My Access portal** and requests the `Contractor Access — Project Engagement` package on behalf of the contractor. They provide the contractor's email address, project name, start date, and expected end date as part of the request questionnaire.
 
The request enters a `Pending approval` state. The business sponsor receives an email notification with a link to the My Access approvals queue. The sponsor has **3 business days** to review and decide. If the sponsor does not respond within 2 days, the request is forwarded to the Identity Governance Administrator as a fallback approver. If no decision is made within the full 3-day window, the request is **automatically denied**.
 
---
 
### 6.2 Provisioning
 
Once the sponsor approves, Entra ID Governance automatically creates a **B2B guest account** for the contractor's external email address (if one does not already exist). All three resource roles — SharePoint Contribute, Teams Member, and application Contributor — are provisioned simultaneously within minutes.
 
The contractor receives an access delivery email with a link to `myaccess.microsoft.com`. On first sign-in, **Conditional Access** intercepts the authentication and requires MFA registration. The contractor sets up Microsoft Authenticator. All subsequent sign-ins require MFA, and no persistent browser session is allowed.
 
---
 
### 6.3 Active Engagement
 
During the engagement, the contractor has access to exactly the three resources specified in the package — nothing else. Their guest account has **no license assigned**, so features like Outlook mailbox and OneDrive storage are unavailable. This is intentional and enforces the minimum footprint principle.
 
Every **90 days (quarterly)**, the sponsor receives an access review notification and certifies each contractor still on the project. If the sponsor does not respond within the review window, **all contractor assignments are automatically removed** — the safer default for external identities.
 
---
 
### 6.4 Engagement End
 
At **14 days** and again at **1 day** before the 180-day expiry, the contractor and sponsor receive email notifications. The manager may request an extension through My Access, which requires sponsor approval with the same workflow as the initial request.
 
If no extension is requested, the assignment **automatically expires** on the end date. The sequence is fully automated:
 
```
Assignment expires
    → All resource roles immediately revoked
    → Guest account sign-in blocked
    → 30 days later: guest account permanently deleted
    → Every step logged in Entra ID audit log (EntitlementManagement category)
```
 
No manual administrator action is required at any point in the offboarding process.
 
---
 
## 7. Design Decisions and Rationale
 
| Decision | Rationale | Trade-off Accepted |
|---|---|---|
| **Manager requests on behalf — not self-service** | Contractors should not discover and request internal resources autonomously. A manager request ensures there is a named internal owner for every contractor engagement. | Adds one step for the manager — minor friction accepted for control. |
| **Single-stage approval by named sponsor** | Sponsors have direct knowledge of the engagement scope. Using Manager as approver would fail for external identities (no manager attribute in Entra ID). Named sponsor is most reliable. | If the sponsor is unavailable, the fallback (IGA admin) must step in — documented and acceptable. |
| **180-day default expiry (not 365)** | Contractor engagements are typically project-based and shorter than a year. 180 days forces a checkpoint without being disruptive for longer engagements that can extend. | Sponsors of long engagements must approve extensions — adds process but ensures active human attestation. |
| **Quarterly review with auto-remove on no response** | External identities have no ongoing HR relationship to trigger offboarding. Quarterly review ensures a human certifies the contractor is still engaged. Auto-remove on inaction protects against orphaned access. | Increases sponsor workload — mitigated by clear email notifications and a simple one-click review interface. |
| **No license assignment to guest accounts** | Contractors need project access only — not Microsoft 365 services. Omitting licenses reduces cost, limits the contractor's footprint, and prevents unintended data access via Outlook/OneDrive. | Contractors cannot use licensed features — must be communicated during onboarding. |
| **Automatic guest account deletion at 30 days** | Ensures no dormant guest accounts accumulate in the directory. Removes all identity data for the contractor after access ends. | If the contractor is re-engaged later, a new guest account must be created — acceptable and ensures a clean start. |
 
---
 
*Document prepared as a portfolio artifact for the Final IAM Project — Day 4.*


















