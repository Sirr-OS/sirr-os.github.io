---
layout: post
title: phosh-pam-duress Installation Guide
subtitle: Setting up emergency authentication protocols on Sirr OS
header-img: img/in-post/2026-6-21/header-duress.jpg
header-style: text
catalog: true
tags:
  - Security
  - PAM
  - Authentication
  - SirrOS
  - PinePhone

---

## What is a Duress Password and Why Does It Matter?

In scenarios where your device is seized or you are coerced into unlocking it, having a hidden emergency protocol can be the difference between data loss and digital self-defense. A **duress password** is a second authentication credential that appears to succeed just like your real password — but instead triggers silent, user-defined actions in the background.

**phosh-pam-duress** brings this critical capability to [Sirr OS](https://github.com/Sirr-os) on the PinePhone Pro.

The power lies in what happens next: while your coercer believes they have normal access, you can be executing scripts that:

* Wipe sensitive encryption keys
* Close encrypted network tunnels
* Send an alert to a trusted contact
* Record evidence (audio or photo)
* Remove the duress module itself, leaving no trace

No suspicious dialogs. No visible delays. Authentication simply succeeds.

---

## How phosh-pam-duress Works

**phosh-pam-duress** is a Pluggable Authentication Module (PAM) that intercepts login attempts and adds duress detection to your system's authentication chain.

When you enter a password at the login prompt:

1. The standard `pam_unix.so` module checks your real password first
2. If it fails, `pam_duress.so` takes over and scans your duress scripts
3. Each script has been cryptographically signed with a unique password hash
4. If your input matches any signed script, that script executes silently as a background process
5. Authentication succeeds — the system appears unlocked normally
6. The script runs with either root privileges (for global scripts) or user privileges (for personal scripts)

The key insight: **the attacker sees nothing unusual**. Your duress password behaves identically to your real password from their perspective.

---

## Installation from Releases

### Step 1: Download the .deb Package

Navigate to the [phosh-pam-duress releases page](https://github.com/Sirr-os/phosh-pam-duress/releases) and download the latest `.deb` file for `arm64`:

```bash
wget https://github.com/Sirr-os/phosh-pam-duress/releases/download/v1.X.X/phosh-pam-duress_1.X.X_arm64.deb
```

### Step 2: Install the Package

```bash
sudo dpkg -i phosh-pam-duress_*_arm64.deb
```

This installs:
* The PAM module (`pam_duress.so`) to `/lib/security/`
* The signing utility (`duress_sign`) to `/usr/bin/`
* A sample wipe script to `/etc/duress.d/`
* Required system directories

### Step 3: Verify Installation

```bash
# Check that the PAM module was installed
ls -la /lib/security/pam_duress.so

# Verify the signing utility is available
which duress_sign

# Confirm the duress directories exist
ls -la /etc/duress.d/
```

---

## Article in progress

---

**Privacy is a right. Take it back — in your pocket.**
