---
layout: post
title: Luks and SirrLuks
subtitle: LUKS Disk Encryption Setup and the SirrLuks Tool
header-img: img/in-post/2026-1-2/header-luks.jpg
header-style: text
catalog: true
tags:
  - Luks
  - Linux
  - Privacy
---

## LUKS Encryption in SirrOS

SirrOS implements full disk encryption using LUKS (Linux Unified Key Setup) to protect your data on the PinePhone Pro. LUKS is an industry-standard encryption method that uses strong cryptographic algorithms to secure the entire filesystem. By default, SirrOS sets up the root filesystem with a default LUKS password, which is **s1rr0s**.

The encryption parameters used in SirrOS are designed for strong security:

* LUKS version 2 for modern key management.

* AES-XTS-ESSIV with SHA-256 for encryption and block scrambling.

* 512-bit key size to resist brute-force attacks.

* Argon2id PBKDF with high memory usage and iteration time to slow down password guessing.

* Compatible with ext4, f2fs, and btrfs filesystems.

These settings ensure that even if someone gains physical access to the device, the encrypted data cannot be read without the correct password.

## Why Change the Default Password

Using the default password **s1rr0s** is extremely risky:

1. Anyone who knows the default password can unlock your encrypted disk.

2. Leaving the default password exposes all sensitive data stored on the device.

3. Default passwords are a common target for automated attacks or social engineering.

Changing the password immediately after installing SirrOS is strongly recommended to maintain data confidentiality.


## SirrLuks: Change LUKS Passphrase	

To make password changes easy, SirrOS provides SirrLuks, a command-line tool that allows you to:

* Add a new password to the LUKS keyslot.

* Remove the old password.

* Ensure the new password matches confirmation input to prevent mistakes.

* Safely manage the encrypted device while it is mounted.

### Example Usage
To run SirrLuks, you can use the tool from the Phosh menu or from the CLI using this command:
```sh
sudo sirrluks
```

The tool will interactively prompt you for:
```sh
Current password: #The default Password or current password
New password:  # New password
Confirm new password: # Confirm password
```

Once executed, SirrLuks securely adds the new key and removes the old one, reporting the status:
```sh
Adding new LUKS key...
Removing old LUKS key...
LUKS password successfully changed
```

## Summary

Using LUKS encryption with secure parameters provides strong data protection for your PinePhone Pro running SirrOS. Changing the default password with SirrLuks ensures that your encrypted data remains private and inaccessible to anyone who might know the default credentials. Always replace the default password before storing sensitive information on the device.
