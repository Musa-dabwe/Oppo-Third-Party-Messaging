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

## Step-by-Step Fix

### 1. Connect your phone via ADB
```bash
adb devices
```

### 2. Identify the security packages
First, list all OPPO/ColorOS security-related packages:
```bash
adb shell pm list packages | grep -i "safe\|security\|guard\|protect"
```

### 3. Disable ALL the culprit packages
**IMPORTANT:** You need to disable ALL of these packages. They work together, so disabling just one or a few won't fix the problem. Run these commands one by one:

```bash
# Disable SecurePay
adb shell pm disable-user --user 0 com.coloros.securepay

# Disable Phone Manager
adb shell pm disable-user --user 0 com.coloros.phonemanager

# Disable Security Guard
adb shell pm disable-user --user 0 com.coloros.securityguard

# Disable Smart Lock
adb shell pm disable-user --user 0 com.coloros.smartlock

# Disable the default SMS app (forces system to use Google Messages only)
adb shell pm disable-user --user 0 com.android.mms

# Disable OPPO SafeCenter (the main culprit)
adb shell pm disable-user --user 0 com.oplus.safecenter

# Disable Security Permission
adb shell pm disable-user --user 0 com.oplus.securitypermission

# Disable Security Keyboard
adb shell pm disable-user --user 0 com.oplus.securitykeyboard

# Disable Quality Protect
adb shell pm disable-user --user 0 com.oplus.qualityprotect

# Disable Family Guard
adb shell pm disable-user --user 0 com.coloros.familyguard

# Disable Remote Guard Service
adb shell pm disable-user --user 0 com.coloros.remoteguardservice
```

**Note:** These packages work together as a security system. If you re-enable any of them (except com.android.mms), the password prompt will likely come back!

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

For example:
```bash
adb shell pm enable com.oplus.safecenter
```

---

## Final Remarks

**TAKE THAT OPPO** 😂

Turns out it wasn't just **com.oplus.safecenter** acting alone - it was a whole gang of OPPO security packages working together like some kind of overprotective security cartel. They all had to go down together.

**Important:** Don't try to re-enable individual packages thinking "oh I only need this one disabled." The password prompt will come right back because these packages are like a hydra - disable one head and the others keep working. You gotta disable them ALL (except com.android.mms which you can re-enable if needed).

Now you can finally send messages in peace without that annoying prompt every single time. Enjoy your newly liberated phone!
