# Selective Wipe

How to remotely wipe **only corporate data** from a MAM-enrolled user's device — without touching their personal files.

> ⚠️ A selective wipe removes **org data protected by app protection policies** (e.g., work profile in Edge). It does **not** factory-reset the device or delete personal files.

---

## When to Use a Selective Wipe

- Employee leaves the company
- Device is lost or stolen
- User moves to a different role that no longer needs access
- Security incident on an unmanaged device
- User requests removal of work data from their personal device

---

## Steps to Perform a Selective Wipe

### Option 1: Wipe by User

Use this when you want to wipe org data from **all** of a user's devices at once.

1. Open **Microsoft Intune admin center** → [https://intune.microsoft.com](https://intune.microsoft.com)
2. Navigate to **Apps → App protection policies → Monitor → App protection status**
3. Click **Assigned users** to see all users with app protection policies applied
4. Find and select the **target user**
5. Click **Wipe data** in the top toolbar
6. Choose which app protection policies to wipe:
   - **All apps** — wipes org data from every protected app
   - **Specific app** — wipes org data from a single app (e.g., Microsoft Edge only)
7. Check the **Also revoke the user's Intune license** box if the user is leaving the company (optional)
8. Click **Yes** to confirm

### Option 2: Wipe by Device (specific device for a user)

Use this when a user has multiple devices and you only want to wipe one.

1. Open **Microsoft Intune admin center** → [https://intune.microsoft.com](https://intune.microsoft.com)
2. Navigate to **Apps → App protection policies → Monitor → App protection status**
3. Click **User report**
4. Search for and select the **target user**
5. You will see a list of the user's **devices with app protection status**
6. Select the **specific device** you want to wipe
7. Click **Wipe data** in the top toolbar
8. Confirm by clicking **Yes**

---

## What Happens After a Selective Wipe

| What gets removed | What stays untouched |
|---|---|
| Org data in Edge work profile (cached pages, bookmarks, cookies) | Personal browsing data in Edge |
| Work account sign-in session | Personal Microsoft account |
| Cached corporate documents | Personal files on the device |
| App protection policy assignments for that device | The Edge browser itself (not uninstalled) |

---

## How to Verify the Wipe Was Successful

1. Go to **Intune admin center → Apps → App protection policies → Monitor → App protection status**
2. Select **User report** and search for the user
3. The device should show a status of **Wiped** or no longer appear in the list
4. On the user's device, the Edge work profile will be removed — if they open Edge, they will no longer see org data

---

## Important Notes

- Selective wipe **only affects MAM-managed data**. It does not perform a full device wipe.
- The wipe command is queued and executes the **next time the device checks in**. If the device is offline, it will process when it reconnects.
- After a wipe, the user can re-enroll in MAM by signing back into Edge with their work account (if their account is still active and policies still target them).
- For users who have left the company, combine the selective wipe with **disabling their Entra ID account** to prevent re-enrollment.
