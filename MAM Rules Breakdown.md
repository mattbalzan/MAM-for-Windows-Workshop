# MAM Settings/Policy/Rules Breakdown

<br>


## Step 1 — Block BYOD Enrollment

| Setting | Value | What it means |
|---|---|---|
| Platform | Windows | This rule only applies to Windows devices |
| Personally owned | **Block** | Users cannot enroll their personal devices into Intune. This is critical — if they enroll, full MDM kicks in and MAM won't work. |
<br>


## Step 2 — Windows Security Center Connector

| Setting | Value | What it means |
|---|---|---|
| Connector | Windows Security Center | Lets Intune read basic security signals from the device (e.g., is antivirus on?). It's lightweight — not full device management. |
<br>


## Step 3 — App Protection Policy

### Data Protection

| Setting | Value | What it means |
|---|---|---|
| Receive data from | All sources | Users **can** paste/import data from personal apps **into** the work profile in Edge. (Lets them bring data in.) |
| Send org data to | No destinations | Users **cannot** move company data **out** of Edge's work profile to any personal app. (Data stays locked in.) |
| Allow cut, copy, paste for | Org data destinations and org data sources | Copy/paste only works **between org apps** (e.g., org Outlook → org Word in Edge). Pasting to Notepad or personal apps is blocked. |
| Print org data | Block | Users cannot print anything from the work profile. No PDF, no physical printer. |
<br>

### Health Checks

| Setting | Value | What it means |
|---|---|---|
| Offline grace period | 1440 min → Block Access | If the device is offline for **24 hours**, block access to org data until it reconnects. This ensures policies stay current. |
| Offline grace period | 90 days → Wipe data | If the device is offline for **90 days**, wipe all org data from Edge. The device is likely lost or abandoned at that point. |
| Disabled account | Block access | If the user's Entra ID account is disabled (e.g., they left the company), immediately block access to org data. |

### Assignment

| Setting | Value | What it means |
|---|---|---|
| Assignments | MAM Users | The policy only applies to users in this Entra ID group — not to devices. MAM is always user-targeted. |
<br>


## Step 4 — Conditional Access Policies

### CA Policy 1: MAM - GRANT require App Protection Policy for Unmanaged Devices

| Setting | Value | What it means |
|---|---|---|
| Users | MAM Users | Applies to your MAM user group |
| Exclude | Admins | Admins are exempt so they don't lock themselves out |
| Cloud apps | Office 365 | Targets all Office 365 apps (SharePoint, Outlook web, etc.) |
| Device platform | Windows | Only fires on Windows devices |
| Client apps | Browser | This policy only evaluates **browser** access |
| Filter for devices | Exclude: `device.trustType -ne "AzureAD" -and device.trustType -ne "ServerAD"` | Corporate/managed devices are **excluded** from this block. The policy only hits personal/unmanaged devices. |
| Grant | Require app protection policy | The browser must have a MAM policy applied — only Edge with MAM satisfies this. Chrome/Firefox will be blocked. |
| Enable policy | On | Policy is active (not in report-only mode) |

```
This policy says: When someone in the MAM Users group tries to access any cloud app 
using desktop clients (like Outlook, Teams), mobile apps, ActiveSync, or other clients
on a non-corporate Windows device, they must have a compliant (Intune-enrolled) device. 
Since BYOD enrollment is blocked in Step 1, personal devices can never satisfy this requirement.

Summary: All native apps (Outlook, Teams, etc.) are completely blocked on personal devices.
There's no way around it — the only door in is Edge with MAM from Policy 1.
```
<br>

### CA Policy 2: MAM - BLOCK office desktop apps on unmanaged Windows

| Setting | Value | What it means |
|---|---|---|
| Users | MAM Users | Same user group |
| Exclude | Admins | Same admin exemption |
| Cloud apps | All cloud apps | Broader than Policy 1 — covers **everything**, not just Office 365 |
| Device platform | Windows | Only fires on Windows |
| Client apps | Mobile apps and desktop clients, Exchange ActiveSync, Other clients | Targets **every non-browser client** — Outlook desktop, Teams app, ActiveSync mail, etc. |
| Filter for devices | Exclude: `device.trustType -ne "AzureAD" -and device.trustType -ne "ServerAD"` | Corporate/managed devices are **excluded** from this block. The policy only hits personal/unmanaged devices. |
| Grant | Require device to be marked as compliant | Demands a compliant Intune-enrolled device. Since Step 1 blocks personal enrollment, personal devices **can never** meet this requirement → access denied. |
| Enable policy | On | Policy is active |

```
This policy says: When someone in the MAM Users group tries to access Office 365 
through a browser on an unmanaged Windows device, they must have an app protection policy
applied (i.e., they must be using Edge with MAM). 
It also blocks file downloads from the browser session.

Summary: You can only use Office 365 in a browser if that browser is Edge with MAM protection.
No downloading files either.
```
<br>

## How It All Fits Together

```
Personal Windows device tries to access company data
│
├─ Desktop app (Outlook, Teams, etc.)
│  → Policy 2 → "Show me a compliant device"
│  → Can't enroll (Step 1) → ❌ BLOCKED
│
├─ Chrome / Firefox browser
│  → Policy 1 → "Show me an app protection policy"
│  → No MAM on Chrome → ❌ BLOCKED
│
└─ Edge browser with MAM
   → Policy 1 → App protection policy present ✅
   → App Protection Policy (Step 3) controls what you can do:
        ├─ View SharePoint → ✅
        ├─ Copy between org apps → ✅
        ├─ Copy to Notepad → ❌
        ├─ Print → ❌
        └─ Download → ❌
```
🥳
