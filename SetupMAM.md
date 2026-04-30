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

**`Intune admin center` → `Devices` → `Enrollment` → `Device platform restrictions`**

*   Edit **Default** or create a scoped policy
*   Platform: **Windows**
*   Personally owned: **Block**

<img height="350" alt="image" src="https://github.com/user-attachments/assets/4d9f1bce-2939-406c-a9d1-fbc1af74e24e" />


✅ Result:  
Users stay **unmanaged**, which is required for Windows MAM to apply.

***

## 2️⃣ Enable Windows Security Center connector

This allows **lightweight health signals** (not compliance).

**`Intune admin center` → `Tenant administration` → `Connectors and tokens` → `Mobile Threat Defense`**

*   Add **Windows Security Center** connector
*   Status may show *Unavailable* until first use — this is expected

<img height="350" alt="image" src="https://github.com/user-attachments/assets/379e8cce-9889-404b-999b-961f83cab50f" />


✅ Result:  
Edge MAM can consume basic Windows Security signals where supported.

***

## 3️⃣ Create the **Windows App Protection Policy**

This is the **core** of MAM for Windows.

**`Intune admin center` → `Apps` → `App protection policies` → `Create policy`**

*   Platform: **Windows**
*   App: **Microsoft Edge**
*   Target: **MAM Users** (not devices)

Configure (example baseline):

*   Data Protection:
    *   Receive data from : `No sources`
    *   Send org data to: `No destinations`
    *   Allow cut, copy and paste for: `Org data destinations and org data sources`
    *   Print org data: `Block`

*   Health Checks:
    | Setting               | Value             | Action |
    | --------------------- | ------------------- | ------------------- |
    | Offline grace period  | 1440 | Block Access (minutes) |
    | Offline grace period | 90 | Wipe data (days) |
    | Disabled account  | | Block access |
    | Max allowed device threat level  | Secured | Block access |
*   Assignments: `MAM Users`

<img height="450" alt="image" src="https://github.com/user-attachments/assets/505d701e-c43d-412e-994b-f7fd58fd431d" />


✅ Result:  
Corporate data is **contained inside Edge’s work profile**

***

## 4️⃣ Create Conditional Access – **Block everything except Edge**

This is not optional.

***

### CA Policy:  ``MAM - GRANT Edge CA``

**`Microsoft Entra admin center` → `Conditional Access` → `New policy`**

*   Assignments:
    *   Users and Groups: `MAM Users`
    *   Exclude: `Break Glass Admins`
*   Target resources:
    *   Cloud apps: `Office 365`
*   Conditions:
    *   Device platform: `Windows`
    *   Client apps: `Browser`
    *   Filter for devices: Exclude rule `device.trustType -eq "AzureAD" -and device.trustType -eq "ServerAD"`
*   Grant:
    *   Grant Access: **Require app protection policy**
*   Enable policy:
    *   `On`

✅ Result:  
Unmanaged devices **fail this policy**, blocking Outlook, Teams, etc.

***

### 5️⃣ Edge Configuration policy: ``Enable Protected Downloads``

**`admin.microsoft.com` → `Settings` → `Microsoft Edge`** (requires Az Role: `Edge Administrator`)

Create a Configuration Policy
* Create a new Configuration Policy targeted to the intended MAM users:

   * Ensure Windows 10+ is selected for platforms
   * Set policy type to Cloud
   * No additional settings need to be added when creating the policy
   * Under Assignments, select the same user group receiving your MAM/CA policies (MAM Users)

Enable Protected Downloads
* After the Configuration Policy has been created and saved, navigate within it to:

**`Customization Settings` → `Security Settings` → `Protected Downloads` → `Enable`**

Important: This setting is only available after the policy is created and saved. You cannot configure it during initial creation. You must go back into the saved policy to find the Customization Settings section.
***

## 5️⃣ End‑user sign‑in flow (what *must* happen)

When the user signs in to Edge:

1. Signs in with **work account**
2. When prompted `Sign in to all apps and websites on this device?` → ✅ **Yes**
3. When prompted `Allow your org to manage your device?` → ✅ **No**

   ⚠️ If the user allows device management, **MAM will not apply**.
<br> 
<img height="200" alt="image" src="https://github.com/user-attachments/assets/b0f7b6aa-c4ac-434a-a4ef-ec260638af87" />

<br> 

<img height="200" alt="image" src="https://github.com/user-attachments/assets/9f57bbda-60b6-48de-9394-4ffe287aa4a2" />
<br>

4. Click **OK** to sync data

<br>  
<img height="200" alt="image" src="https://github.com/user-attachments/assets/58ed7bb3-6e28-4280-a9bb-7f6963d9979e" />
<br>  

5. Edge Profile to access org resources will be setup.

***

## 6️⃣ MAM Validation

On the test device:

*   Outlook desktop → ❌ blocked
*   Teams desktop → ❌ blocked
*   Edge (work profile):
    *   SharePoint → ✅ allowed
    *   Copy text from org Outlook → org Word ✅ allowed
    *   Copy text → ❌ blocked to Notepad
    *   Download file from Outlook org → ❌ blocked
    *   Send file to OneDrive <ORG> from Outlook org → ✅ allowed (file syncs to OD4B Attachments folder)
    *   Screen grab or snipping tool → ❌ blocked with black screen
    *   Print from org → ❌ blocked

✅ This confirms Edge MAM is active.

***

## ✅ Troubleshooting Checklist

✅ Check Edge & MAM Policies have landed
*  Open new **Edge** tab → Enter URL: `Edge://edge-dlp-internals`
*  Open the **MamLog.txt** file → `%temp%\Microsoft\Edge\User Data`
*  Open the **MamCache.json** in same folder.

✅ If Downloads Are Blocked Entirely (No OD4B Redirect)
* Missing Edge Management Policy: The most common issue. Verify that the Configuration Policy in admin.microsoft.com exists, targets the correct users, and has "Protected Downloads" enabled under Customization Settings → Security Settings
* Policy not yet saved: The "Protected Downloads" toggle is only visible after the Configuration Policy is created and saved. Go back into the saved policy to enable it
* User group mismatch: The Intune APP and Edge Management Configuration Policy must target the same users. Check group membership in both portals
* Policy propagation delay: Allow up to 4-8 hours for policy to propagate. User can force sync by signing out and back into Edge

✅ If Downloads Go to Local Disk Instead of OD4B
* APP not set to "No destinations": If Send org data to is set to All destinations, downloads proceed normally and Protected Downloads never activates. Must be No destinations
* OneDrive sync client not signed in: The user's OneDrive sync client must be signed in with their corporate account. Without it, there's no local OD4B path for Edge to redirect to
* Device is MDM-enrolled: MAM policies do not apply on enrolled devices. If the device is Entra ID Joined or MDM managed, the APP is not enforced

✅ If the "Microsoft Edge Downloads" Folder Doesn't Appear in OD4B
* First download triggers creation: The folder is created automatically on the first Protected Download. It won't exist until a user actually downloads a file
* OD4B quota: Verify the user has sufficient OD4B storage quota
* Sync conflicts: If OneDrive sync is paused or encountering errors, files may queue but not appear. Check sync client status
