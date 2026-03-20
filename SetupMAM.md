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

## 3️⃣ Create the **Windows App Protection Policy** (this is MAM)

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
    | --------------------- | ------------------- |  ------------------- |
    | Offline grace period  | 1440 | Block Access (minutes) |
    | Offline grace period | 90    | Wipe data (days) |
    | Disabled account  | | Block access |
*   Assignments: `MAM Users`
 
<img height="450" alt="image" src="https://github.com/user-attachments/assets/ce977cdb-919f-4193-b0f2-def51e818c6b" />


✅ Result:  
Corporate data is **contained inside Edge’s work profile**

***

## 4️⃣ Create Conditional Access – **Block everything except Edge**

You need **two CA policies**. This is not optional.

***

### CA Policy 1:  MAM -Block desktop & mobile apps on unmanaged Windows

**Microsoft Entra admin center → Conditional Access → New policy**

*   Assignments:
    *   Users and Groups: `MAM Users`
*   Target resources:
    *   Cloud apps: `Office 365`
*   Conditions:
    *   Device platform: `Windows`
    *   Client apps: `Browser`
*   Grant:
    *   **Require app protection policy**
*   Session:
    *   Use Conditional Access App Control: `Block downloads (Preview)`
*   Enable policy:
    *   `On`

✅ Result:  
Unmanaged devices **fail this policy**, blocking Outlook, Teams, etc.

***

### CA Policy 2: MAM - Allow browser access only via Edge

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
    *   **Require device to be marked as compliant**
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
    *   ✅ **Uncheck** “Allow my organisation to manage my device”
    *   ✅ Continue

⚠️ If the user allows device management, **MAM will not apply**

***

## 6️⃣ Validation (how to prove it works)

On the test device:

*   Outlook desktop → ❌ blocked
*   Teams desktop → ❌ blocked
*   Edge (work profile):
    *   SharePoint → ✅ allowed
    *   Copy text → ❌ blocked to Notepad
    *   Download file → ❌ blocked or redirected

✅ This confirms Edge MAM is active.

<img height="350" alt="image" src="https://github.com/user-attachments/assets/e6fcf141-7382-4d3b-9069-dc7bc6e98563" />

<img height="350" alt="image" src="https://github.com/user-attachments/assets/347623e3-4caf-48a9-aeb0-f7e30a8f70e6" />

<img height="350" alt="image" src="https://github.com/user-attachments/assets/210ded78-c09e-4180-8447-fdca06d33da4" />



# ✅ POC summary

| Layer                 | Purpose             |
| --------------------- | ------------------- |
| App Protection Policy | Data containment    |
| Conditional Access    | Enforcement         |
| Edge work profile     | Identity separation |
| No MDM                | BYOD‑safe           |
