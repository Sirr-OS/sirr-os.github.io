---
layout: post
title: Duress Password Installation Guide
subtitle: Setting up emergency authentication protocols on Sirr OS
header-img: img/in-post/2026-6-21/header-duress.jpg
header-style: text
catalog: true
tags:
  - Security
  - PAM
  - Authentication
  - SirrOS
  - PinePhone Pro

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
* A sample LUKS wipe script to `/etc/duress.d/`
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

## PAM Configuration (Automatic)

The `.deb` package automatically configures PAM during installation. The `/etc/pam.d/common-auth` file is updated to integrate `pam_duress.so` into the authentication chain.

To verify the configuration is correct, check:

```bash
sudo grep -A2 "pam_unix.so" /etc/pam.d/common-auth
```

You should see:

```
auth    [success=2 default=ignore]      pam_unix.so
auth    [success=1 default=ignore]      pam_duress.so
auth    requisite                       pam_deny.so
```

If the configuration is missing or incorrect, you can manually edit it:

```bash
sudo nano /etc/pam.d/common-auth
```

---

## Creating Your Duress Scripts

### Example: LUKS Header Destruction Script

The package includes a sample script. Here's how it works:

```bash
#!/bin/bash
# Remove all key slots from active volumes
cryptsetup erase /dev/mmcblk1p2 --batch-mode

# Remove LUKS header
cryptsetup luksHeaderWipe /dev/mmcblk1p2 --batch-mode

# Overwrite the beginning of the partition with random data
dd if=/dev/urandom of=/dev/mmcblk1p2 bs=512 count=8192 conv=notrunc

# Power off the system
systemctl poweroff
```

**Important:** You must customize `/dev/mmcblk1p2` to match your actual encrypted partition. Use `lsblk` to identify your device:

```bash
lsblk -o NAME,TYPE,FSTYPE
```

Save your personalized script as `~/.duress/wipe_luks.sh` and set permissions:

```bash
chmod 500 ~/.duress/wipe_luks.sh
```

---

## Signing Your Duress Scripts

Once your script is ready with correct permissions, sign it with a duress password:

```bash
duress_sign ~/.duress/wipe_luks.sh
```

You'll be prompted to enter your duress password twice:

```
Password: [will not echo]
Confirm: 
Reading /home/user/.duress/wipe_luks.sh...
Done.
6B8B621EFB8050B83AAC734D56BF9165DC55D709CBAD530C6241E8A352587B3F
```

This creates a `.sha256` signature file:

```bash
~/.duress/wipe_luks.sh.sha256
```

When you enter this duress password at the login screen, the PAM module will detect it, execute the script silently, and grant access.

---

## Understanding Script Customization

Every duress script must be tailored to your specific device and threat model. Here are the key areas you'll need to personalize:

### Device Identification

Different PinePhone configurations may have different partition layouts. Always verify your encrypted partition before creating scripts:

```bash
# List all block devices
lsblk -o NAME,TYPE,FSTYPE,SIZE

# Check current mountpoints
mount | grep crypt
```

### Privilege Levels

Scripts can execute as either:

* **User-level** (`~/.duress/`): Runs with your user permissions. Good for personal data cleanup or user-specific alerts.
* **System-level** (`/etc/duress.d/`): Runs as root. Required for LUKS operations, system-wide wipes, or affecting all users.

### Post-Execution Behavior

Consider what should happen after your script runs:

* **Immediate shutdown** — prevents re-entry and observation of filesystem state
* **Silent return to login** — device appears to work normally (attacker may not notice anything unusual)
* **Alert mechanisms** — notify trusted contacts via secure channels before shutdown
* **Evidence preservation** — log the event before wiping

---

## Beyond LUKS: Other Duress Scenarios

The LUKS example is just one use case. Depending on your threat model, you might create scripts for:

### Network Isolation

Sever all network connections, close VPN tunnels, or disable wireless entirely. This prevents remote monitoring or data exfiltration.

### Credential Removal

Beyond encryption keys, remove SSH keys, API credentials stored in config files, or authentication tokens from running services. This denies attackers access to your external accounts.

### Evidence Collection

Before any wipe occurs, trigger automated recording. Capture audio, take photos via the camera, or record the device's location. This creates an auditable record of the coercion event.

### Selective Data Deletion

Instead of full device destruction, selectively wipe specific directories containing sensitive information while leaving the device operational. The attacker sees a functioning system but finds nothing of value.

### Blockchain/Web3 Defense

For users managing crypto wallets or blockchain keys, trigger the removal of signing keys, transaction history, or connected dapps — preventing unauthorized transactions.

---

## Best Practices

* **Test scripts on identical hardware** — device paths and filesystem layouts vary
* **Use absolute paths in scripts** — avoid relying on `$PATH` or environment variables
* **Keep duress passwords unique** — never reuse them across devices or accounts
* **Document your threat model** — write down exactly what scenario you're defending against
* **Backup your scripts** — store encrypted copies on external media for recovery
* **Never share duress passwords** — they're single-use credentials tied to your threat scenario
* **Verify permissions are correct** — only `500`, `540`, or `550` permission masks are allowed

---

## Testing and Safety

**Before deploying duress scripts on your main device:**

1. Create a backup of your LUKS header or important data
2. Test scripts on a spare device with non-critical data
3. Verify the signed password actually triggers the script
4. Ensure the script behaves as expected without causing data loss

If anything goes wrong, restore from your backup.

---

## Removing or Updating Scripts

To remove a duress script and its signature:

```bash
rm ~/.duress/wipe.sh
rm ~/.duress/wipe.sh.sha256
```

To update a script with a new duress password, simply sign it again with `duress_sign` — the old signature becomes invalid.

---

**Privacy is a right. Take it back — in your pocket.**
