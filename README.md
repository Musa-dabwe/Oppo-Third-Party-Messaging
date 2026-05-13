# Oppo-Third-Party-Messaging
Disabling Oppo third party messaging password prompt when trying to send an sms using ADB.

# How to Fix Annoying OPPO Password Prompts When Sending Messages

## Problem
On OPPO/Realme devices (ColorOS), you get an annoying password prompt every time you try to send a message using a third party messaging app such as Google Messages, even after setting Google Messages as your default SMS app and logging out of your OPPO account.

## Solution
The password prompt is caused by OPPO's security packages running in the background. You need to disable them using ADB.

## Prerequisites
- ADB installed on your computer
- If PC not available you can use wireless debugging [Tutorial](https://www.youtube.com/watch?v=CupHemQJu_g&t=97s)
- USB debugging enabled on your phone
- USB cable to connect phone to computer

## Package Research

Not all security packages are equally responsible for the SMS password prompt. Below is a breakdown of each known OPPO/ColorOS security package, what it actually does, and how likely it is to be the culprit based on user reports.

### High Priority (Most Likely Culprits)

| Package | Description | Why It's a Culprit |
|---|---|---|
| `com.oplus.safecenter` | **Safe Center** — main security hub: app lock, virus scan, Find My Device, privacy permissions, payment protection | The primary gatekeeper that checks permissions before allowing sensitive actions like SMS sending |
| `com.coloros.securityguard` | **Security Guard** — real-time monitoring, malware scanning, permission checks during sensitive operations | Actively intercepts and prompts for verification when apps try to send SMS |
| `com.oplus.securitypermission` | **Permission Manager** — manages runtime permission prompts, including the "allow app to send SMS?" dialog | Directly controls the SMS permission prompt dialog |

### Medium Priority (Part of the Security Chain)

| Package | Description | Why It's a Culprit |
|---|---|---|
| `com.coloros.securepay` | **Payment Protection** — secures financial transactions and banking apps | May intercept SMS (used for 2FA/banking) but less likely to trigger on regular texts. Not present on all ColorOS versions. |
| `com.coloros.phonemanager` | **Phone Manager / Security Center** — storage cleanup, battery saving, virus scan, app management | Older ColorOS equivalent of safecenter. Not present on newer ColorOS versions (merged into safecenter). |
| `com.oplus.securitykeyboard` | **Secure Keyboard** — a special keyboard that auto-pops up when entering passwords, PINs, or sensitive fields | Unlikely to cause the prompt itself, but disabling it removes the secure input overlay that may interfere |
| `com.oplus.qualityprotect` | **Quality Protect** — app verification & integrity checks when sideloading APKs or performing sensitive operations | May verify app integrity before allowing SMS sending, but less directly involved |

### Low Priority (Safe to Skip Unless Prompt Persists)

| Package | Description | Why It's a Culprit |
|---|---|---|
| `com.coloros.familyguard` | **Family Guard** — parental controls, usage limits, content restrictions | Unlikely to be related to SMS prompts; for family/parental features |
| `com.coloros.remoteguardservice` | **Remote Guard Service** — remote device management, anti-theft, remote lock/wipe | Anti-theft service, unlikely to intercept SMS sending |
| `com.coloros.smartlock` | **Smart Lock** — trusted devices/places/faces to keep phone unlocked | Not related to SMS permissions; handles unlock automation |
| `com.android.mms` | **Android default SMS/MMS app** | Disabling this forces the system to use your third-party SMS app as the only option |
| `com.oplus.eyeprotect` | **Eye Comfort** — blue light filter / eye strain reduction | **Not security-related at all.** Safe to ignore. |

## Step-by-Step Fix

### 1. Connect your phone via ADB
```bash
adb devices
```

### 2. Identify which security packages exist on YOUR device
OPPO changes package names across ColorOS versions. First, check what's actually installed:

```bash
adb shell pm list packages | grep -i "safe\|security\|guard\|protect"
```

Compare the output with the table above to see which packages are present.

### 3. Disable the packages

Start with the **High Priority** packages first — these are the most likely to fix the prompt:

```bash
# Disable OPPO SafeCenter (main culprit — try suspend if disable-user fails)
adb shell pm disable-user --user 0 com.oplus.safecenter

# Disable Security Guard
adb shell pm disable-user --user 0 com.coloros.securityguard

# Disable Security Permission
adb shell pm disable-user --user 0 com.oplus.securitypermission
```

**Note:** On some ColorOS versions, `com.oplus.safecenter` is locked down and `pm disable-user` may be denied. If that happens, use `suspend` instead — it achieves the same effect:
```bash
adb shell pm suspend com.oplus.safecenter
```
Being suspended persists across reboots until you unsuspend it with `adb shell pm unsuspend com.oplus.safecenter`.

If the prompt persists, also disable the **Medium Priority** packages:

```bash
# Disable SecurePay (if present)
adb shell pm disable-user --user 0 com.coloros.securepay

# Disable Phone Manager (if present)
adb shell pm disable-user --user 0 com.coloros.phonemanager

# Disable Security Keyboard
adb shell pm disable-user --user 0 com.oplus.securitykeyboard

# Disable Quality Protect
adb shell pm disable-user --user 0 com.oplus.qualityprotect

# Disable the default SMS app (forces system to use Google Messages only)
adb shell pm disable-user --user 0 com.android.mms
```

If it STILL persists, also disable the remaining:

```bash
# Disable Family Guard
adb shell pm disable-user --user 0 com.coloros.familyguard

# Disable Remote Guard Service
adb shell pm disable-user --user 0 com.coloros.remoteguardservice

# Disable Smart Lock (if present)
adb shell pm disable-user --user 0 com.coloros.smartlock
```

### 4. Reboot your phone
```bash
adb reboot
```

### 5. Test
After reboot, open Google Messages and try sending a message. The password prompt should be **GONE**!

## Re-enabling Packages (If Needed)
If you need to re-enable any package later:
```bash
adb shell pm enable <package-name>
```

If a package was suspended instead of disabled:
```bash
adb shell pm unsuspend <package-name>
```

For example:
```bash
adb shell pm enable com.oplus.safecenter
```

---

## Tested On

| Device | ColorOS Version | Outcome |
|---|---|---|
| OPPO Reno 3 Youth (PCLM50) | ColorOS 12 | Password prompt eliminated after disabling the 3 High Priority packages. `com.oplus.safecenter` required `pm suspend` (was locked down). |

---

## Final Remarks

**TAKE THAT OPPO**

Turns out it wasn't just **com.oplus.safecenter** acting alone — it was a whole gang of OPPO security packages working together like some kind of overprotective security cartel. However, based on research and real-world testing, the three most likely culprits are **safecenter**, **securityguard**, and **securitypermission**. Try those first before nuking everything.

**Note:** Package names vary across ColorOS versions. Some packages listed here may not exist on your device, and you might have new ones not listed. Always run the `pm list packages` command first to see what's actually installed.

**If you disable via `pm suspend` instead of `pm disable-user`:** The package will remain suspended across reboots, but a factory reset or major OTA system update may reset it. You'll know if the prompt comes back — just re-apply the command.

Now you can finally send messages in peace without that annoying prompt every single time. Enjoy your newly liberated phone!
