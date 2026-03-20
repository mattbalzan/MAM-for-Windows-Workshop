>✅ Setup Steps: MAM for Windows (Edge MAM)

## 0️⃣ Prerequisites (MANDATORY)

These are **hard requirements**, not best practices.

*   Microsoft Intune licence assigned to users
*   Microsoft Entra ID P1 or P2 (for Conditional Access)
*   Supported OS:
    *   Windows 11 (recommended), or
    *   Windows 10 20H2+ with required cumulative updates
*   Microsoft Edge (current Stable or Extended Stable)
*   Entra ID Group to target MAM users: `MAM Users` (this is my demo test group)
*   **Device must NOT be Intune‑enrolled** (MDM breaks MAM for Windows)

***

## 1️⃣ Block BYOD users from enrolling devices (important guardrail)

This prevents users bypassing Edge MAM by enrolling their personal device.

**Intune admin center → Devices → Enrollment → Device platform restrictions**

*   Edit **Default** or create a scoped policy
*   Platform: **Windows**
*   Personally owned: **Block**

<img height="350" alt="image" src="https://github.com/user-attachments/assets/4d9f1bce-2939-406c-a9d1-fbc1af74e24e" />


✅ Result:  
Users stay **unmanaged**, which is required for Windows MAM to apply.

***

## 2️⃣ Enable Windows Security Center connector

This allows **lightweight health signals** (not compliance).

**Intune admin center → Tenant administration → Connectors and tokens → Mobile Threat Defense**

*   Add **Windows Security Center** connector
*   Status may show *Unavailable* until first use — this is expected

<img height="350" alt="image" src="https://github.com/user-attachments/assets/379e8cce-9889-404b-999b-961f83cab50f" />


✅ Result:  
Edge MAM can consume basic Windows Security signals where supported.

***

## 3️⃣ Create the **Windows App Protection Policy**

This is the **core** of MAM for Windows.

**Intune admin center → Apps → App protection policies → Create policy**

*   Platform: **Windows**
*   App: **Microsoft Edge**
*   Target: **MAM Users** (not devices)

Configure (example baseline):

*   Data Protection:
    *   Receive data from : `All sources`
    *   Send org data to: `No destinations`
    *   Allow cut, copy and paste for: `Org data destinations and org data sources`
    *   Print org data: `Block`

*   Health Checks:
    | Setting               | Value             | Action |
    | --------------------- | ------------------- | ------------------- |
    | Offline grace period  | 1440 | Block Access (minutes) |
    | Offline grace period | 90 | Wipe data (days) |
    | Disabled account  | | Block access |
*   Assignments: `MAM Users`
 
<img height="450" alt="image" src="https://github.com/user-attachments/assets/ce977cdb-919f-4193-b0f2-def51e818c6b" />


✅ Result:  
Corporate data is **contained inside Edge’s work profile**

***

## 4️⃣ Create Conditional Access – **Block everything except Edge**

This is not optional.

***

### CA Policy 1:  ``MAM - Block desktop & mobile apps on unmanaged Windows``

**Microsoft Entra admin center → Conditional Access → New policy**

*   Assignments:
    *   Users and Groups: `MAM Users`
    *   Exclude: `Admins`
*   Target resources:
    *   Cloud apps: `Office 365`
*   Conditions:
    *   Device platform: `Windows`
    *   Client apps: `Browser`
*   Grant:
    *   Grant Access: **Require app protection policy**
*   Session:
    *   Use Conditional Access App Control: `Block downloads (Preview)`
*   Enable policy:
    *   `On`

✅ Result:  
Unmanaged devices **fail this policy**, blocking Outlook, Teams, etc.

***

### CA Policy 2: ``MAM - Allow browser access only via Edge``

**New Conditional Access policy**

*   Assignments:
    *   Users and Groups: `MAM Users`
    *   Exclude: `Admins`
*   Target resources:
    *   Cloud apps: `All cloud apps`
*   Conditions:
    *   Device platform: `Windows`
    *   Client apps: `Mobile apps and desktop clients`,`Exchange ActiveSync`,`Other clients`
    *   Filter for devices: Exclude rule `device.deviceOwnership -eq "Company"`
*   Grant:
    *   Grant Access: **Require device to be marked as compliant**
*   Enable policy:
    *   `On`

✅ Result:  
Only **Edge with MAM** can access M365 from unmanaged devices

***

## 5️⃣ End‑user sign‑in flow (what *must* happen)

When the user signs in to Edge:

1.  User opens **Edge**
2.  Signs in with **work account**
3.  When prompted:
    *   ✅ **Click** `Yes`
    <img height="200" alt="image" src="https://github.com/user-attachments/assets/b0f7b6aa-c4ac-434a-a4ef-ec260638af87" />

    *   ✅ **Click** `No`
    <img height="200" alt="image" src="https://github.com/user-attachments/assets/9f57bbda-60b6-48de-9394-4ffe287aa4a2" />

⚠️ If the user allows device management, **MAM will not apply**

4. Edge Profile to access org resources will be setup.

   <img height="200" alt="image" src="https://github.com/user-attachments/assets/58ed7bb3-6e28-4280-a9bb-7f6963d9979e" />


***

## 6️⃣ MAM Validation

On the test device:

*   Outlook desktop → ❌ blocked
*   Teams desktop → ❌ blocked
*   Edge (work profile):
    *   SharePoint → ✅ allowed
    *   Copy text from org Outlook → org Word ✅ allowed
    *   Copy text → ❌ blocked to Notepad
    *   Download file → ❌ blocked or redirected
    *   Screen grab or snipping tool → ❌ blocked with black
    *   Print from org → ❌ blocked

✅ This confirms Edge MAM is active.
***

# ✅ Troubleshooting MAM

1.  Open new **Edge** tab.
2.  Enter text: `Edge://edge-dlp-internals`.
3.  Open the MamLog.txt file in `%temp%\Microsoft\Edge\User Data`.
4.  Open the MamCache.json in same folder.


