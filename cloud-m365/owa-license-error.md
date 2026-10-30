# [KB-001] Outlook Web App Error 500: OwaUserHasNoMailboxAndNoLicenseAssignedException

- **Incident ID:** KB-001
- **Severity:** Low (Single User Impact)
- **Category:** SaaS / Identity & Access Management / M365
- **Platform:** Microsoft 365, Exchange Online, Microsoft Entra ID
- **Status:** Resolved

---

## 1. Problem Description
A newly provisioned or modified Microsoft 365 user account is unable to access Outlook Web App (https://outlook.office.com). The browser halts on an error screen with HTTP status code 500.

### Error Signature & Logs:
- UTC Date: 2026-10-30T01:14:52.002Z
- BootResult: configuration
- Back Filled Errors: Unhandled Rejection: Error: 500:undefined|undefined:undefined
- err: Microsoft.Exchange.Clients.Owa2.Server.Core.OwaUserHasNoMailboxAndNoLicenseAssignedException
- esrc: StartupData
- et: ServerError
- st: 500
- ehk: X-OWA-Error

---

## 2. Root Cause Analysis (RCA)
1. **Primary Cause:** The Microsoft Entra ID (Azure AD) user object lacks an active Exchange Online license assignment, preventing the Exchange Mailbox Provisioning Service from generating the user's primary mailbox.
2. **Secondary Failure Mode:** The client browser holds stale authentication session cookies and tokens, causing recurring redirect loops and rendering cached error screens even after administrative license adjustments.

---

## 3. Resolution Steps

### Phase 1: Administrative License Assignment (Admin Side)
1. Sign in to the Microsoft 365 Admin Center (https://admin.microsoft.com) using tenant admin credentials.
2. Navigate to Users > Active users.
3. Select the target user account.
4. Open the Licenses and apps tab.
5. Under Licenses, assign a valid subscription plan that includes Exchange Online (e.g., Microsoft 365 Business, Office 365 Enterprise, or Faculty/Student plan).
6. Under Apps, ensure the checkbox for Exchange Online is explicitly enabled.
7. Click Save changes.
8. Allow up to 15–30 minutes for directory synchronization and backend mailbox database provisioning.

### Phase 2: Client Session Invalidation & Re-authentication (Client Side)
To bypass the cached redirect loop without losing unrelated browsing sessions:
1. Open Google Chrome Settings: chrome://settings/content/all.
2. Filter for domain data: office.com and live.com.
3. Clear stored data and cookies specifically for these Microsoft authentication endpoints.
4. Alternatively, test access via an Incognito window (Ctrl + Shift + N).
5. Access https://outlook.office.com and authenticate with target credentials.

---

## 4. Verification
- The user successfully logs into Outlook on the web without encountering exception traces.
- Incoming and outgoing test emails flow as expected.

---

## 5. Prevention & Hardening
- Implement Group-Based Licensing in Microsoft Entra ID to automatically assign required license suites based on security group membership upon account creation.
- Establish an onboarding verification checklist for IT administrators prior to credential handover.
