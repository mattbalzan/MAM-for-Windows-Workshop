>✅ Setup Steps: MAM for Windows (Edge MAM / MAM‑without‑Enrollment)

## 0️⃣ Prerequisites (MANDATORY)

These are **hard requirements**, not best practices.

*   Microsoft Intune licence assigned to users
*   Microsoft Entra ID P1 or P2 (for Conditional Access)
*   Supported OS:
    *   Windows 11 (recommended), or
    *   Windows 10 20H2+ with required cumulative updates
*   Microsoft Edge (current Stable or Extended Stable)
*   Entra ID User Target test group
*   **Device must NOT be Intune‑enrolled** (MDM breaks MAM for Windows) [\[https://fr...s/original\]](https://fr-prod.asyncgw.teams.microsoft.com/v1/objects/0-eus-d6-67d80315cdf90406d6dc43d502dca379/views/original)

***

## 1️⃣ Block BYOD users from enrolling devices (important guardrail)

This prevents users bypassing Edge MAM by enrolling their personal device.

**Intune admin center → Devices → Enrollment → Device platform restrictions**

*   Edit **Default** or create a scoped policy
*   Platform: **Windows**
*   Personally owned: **Block**

✅ Result:  
Users stay **unmanaged**, which is required for Windows MAM to apply [\[learn.microsoft.com\]](https://learn.microsoft.com/en-us/intune/intune-service/apps/protect-mam-windows)

***

## 2️⃣ Enable Windows Security Center connector

This allows **lightweight health signals** (not compliance).

**Intune admin center → Tenant administration → Connectors and tokens → Mobile Threat Defense**

*   Add **Windows Security Center** connector
*   Status may show *Unavailable* until first use — this is expected

✅ Result:  
Edge MAM can consume basic Windows Security signals where supported [\[https://fr...s/original\]](https://fr-prod.asyncgw.teams.microsoft.com/v1/objects/0-eus-d6-67d80315cdf90406d6dc43d502dca379/views/original)

***

## 3️⃣ Create the **Windows App Protection Policy** (this is MAM)

This is the **core** of MAM for Windows.

**Intune admin center → Apps → App protection policies → Create policy**

*   Platform: **Windows**
*   App: **Microsoft Edge**
*   Target: **User groups** (not devices)

Configure (example baseline):

*   Data transfer:
    *   Block copy/paste to unmanaged apps
    *   Block Save As to local disk
*   Access:
    *   Require PIN (optional, org choice)
*   Conditional launch:
    *   Block on unsupported OS versions

✅ Result:  
Corporate data is **contained inside Edge’s work profile** [\[https://fr...s/original\]](https://fr-prod.asyncgw.teams.microsoft.com/v1/objects/0-eus-d6-67d80315cdf90406d6dc43d502dca379/views/original)

***

## 4️⃣ Create Conditional Access – **Block everything except Edge**

You need **two CA policies**. This is not optional.

***

### CA Policy 1 – Block desktop & mobile apps on unmanaged Windows

**Microsoft Entra admin center → Conditional Access → New policy**

*   Assignments:
    *   Users: Target test group
    *   Cloud apps: Microsoft 365 (or All cloud apps)
*   Conditions:
    *   Device platform: **Windows**
    *   Client apps:
        *   ✅ Mobile apps and desktop clients
        *   ✅ Exchange ActiveSync
*   Grant:
    *   **Require device to be marked as compliant**

✅ Result:  
Unmanaged devices **fail this policy**, blocking Outlook, Teams, etc. [\[learn.microsoft.com\]](https://learn.microsoft.com/en-us/intune/intune-service/apps/protect-mam-windows)

***

### CA Policy 2 – Allow browser access **only via Edge MAM**

**New Conditional Access policy**

*   Assignments:
    *   Same user group
    *   Same cloud apps
*   Conditions:
    *   Device platform: **Windows**
    *   Client apps: **Browser**
*   Grant:
    *   ✅ Require **App Protection Policy**
    *   ✅ (Optional) Also allow compliant devices
*   Session:
    *   (Optional) Defender for Cloud Apps controls

✅ Result:  
Only **Edge with MAM** can access M365 from unmanaged devices [\[learn.microsoft.com\]](https://learn.microsoft.com/en-us/intune/intune-service/apps/protect-mam-windows)

***

## 5️⃣ Configure Edge behavior with App Configuration Policy

This shapes the browser UX.

**Intune admin center → Apps → App configuration policies → Add**

*   Platform: Windows
*   App: Microsoft Edge
*   Examples:
    *   Force work profile separation
    *   Disable personal profile sync
    *   Control extensions

✅ Result:  
Cleaner user experience and fewer support calls [\[andrewstaylor.com\]](https://andrewstaylor.com/2023/08/03/byod-and-mam-for-windows-protecting-your-data-with-intune/)

***

## 6️⃣ End‑user sign‑in flow (what *must* happen)

When the user signs in to Edge:

1.  User opens **Edge**
2.  Signs in with **work account**
3.  When prompted:
    *   ✅ **Uncheck** “Allow my organisation to manage my device”
    *   ✅ Continue

⚠️ If the user allows device management, **MAM will not apply** [\[learn.microsoft.com\]](https://learn.microsoft.com/en-us/intune/intune-service/apps/protect-mam-windows)

***

## 7️⃣ Validation (how to prove it works)

On the test device:

*   Outlook desktop → ❌ blocked
*   Teams desktop → ❌ blocked
*   Edge (work profile):
    *   SharePoint → ✅ allowed
    *   Copy text → ❌ blocked to Notepad
    *   Download file → ❌ blocked or redirected

✅ This confirms Edge MAM is active.

<img width="629" height="502" alt="image" src="https://github.com/user-attachments/assets/e6fcf141-7382-4d3b-9069-dc7bc6e98563" />

<img width="628" height="466" alt="image" src="https://github.com/user-attachments/assets/347623e3-4caf-48a9-aeb0-f7e30a8f70e6" />

<img width="1049" height="668" alt="image" src="https://github.com/user-attachments/assets/210ded78-c09e-4180-8447-fdca06d33da4" />



# ✅ POC summary

| Layer                 | Purpose             |
| --------------------- | ------------------- |
| App Protection Policy | Data containment    |
| Conditional Access    | Enforcement         |
| Edge work profile     | Identity separation |
| No MDM                | BYOD‑safe           |
