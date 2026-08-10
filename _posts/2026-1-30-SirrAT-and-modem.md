---
layout: post
title: SirrAT and Mobile Identity
subtitle: Using SirrAT for Modem Control and Understanding IMEI, IMSI, and BTS Risks
header-img: img/in-post/2026-1-30/header-sirrat.jpg
header-style: text
catalog: true
tags:
  - SirrAT
  - Modems
  - Privacy
  - IMEI-IMSI
  - PinePhone Pro

---

## SirrAT Overview

SirrAT is a tool included in Sirr OS designed to interact with cellular modems using AT commands.  
It acts as a user-friendly interface for sending commands to a modem through a serial interface, relying internally on tools such as `atinout`.

SirrAT is primarily intended for diagnostics, inspection, and controlled configuration of modem parameters on Linux-based mobile devices such as the PinePhone Pro.

Typical use cases include:

- Querying modem status and network registration
- Monitoring signal strength in real-time
- Inspecting Base Transceiver Station (BTS) information
- Sending standard AT commands without manually handling serial devices
- Providing a safer abstraction than raw `echo > /dev/ttyUSB*`

SirrAT does not bypass the mobile network, the operator, or legal constraints. It simply allows direct interaction with the modem firmware.

---

## Running SirrAT

### From the Graphical Menu

SirrAT can be launched from the Phosh menu on your PinePhone Pro. Look for the SirrAT icon in your applications menu, or search for "SirrAT" in the app launcher. The GUI will automatically detect available USB modem ports.

### From the Terminal

Open a terminal and run:

```bash
sirrat
```

SirrAT will automatically detect all available USB modem ports (typically `/dev/ttyUSB0`, `/dev/ttyUSB1`, etc.) and prompt you to select which one to use.

If administrative privileges are required (which is typical for direct modem access), SirrAT will request them through PolicyKit or `sudo`, depending on your system configuration.

---

## Port Detection and Selection

When you launch SirrAT, it immediately scans for connected USB modem ports:

```
[*] Detecting available USB modem ports...
[*] Detected ports:
    [0] /dev/ttyUSB0
    [1] /dev/ttyUSB1
[?] Select port number: _
```

On a PinePhone Pro with a single modem, you'll typically see one or two USB ports. Select the correct one to proceed. If no ports are detected, verify your modem is properly connected and recognized by the system.

---

## SirrAT Main Menu

Once a port is selected, SirrAT presents an interactive menu with the following options:

```
[*] Options menu
    1) Show IMEI (AT+GSN)
    2) Show IMSI (AT+CIMI)
    3) Show signal strength (AT+CSQ)
    4) Real-time signal monitor
    5) Show BTS antennas
    6) Change IMEI (advanced)
    7) Reboot modem
    8) Exit
[?] Select option: _
```

Each option corresponds to specific AT commands sent to your modem. Below is a detailed explanation of each.

---

## Option 1: Show IMEI

**Command:** `AT+GSN`

Displays the **International Mobile Equipment Identity** — the unique hardware identifier of your device.

```
[*] Sending AT command: AT+GSN
357687091234567
```

**What it tells you:**
- Confirms your modem is responding
- Verifies the device identifier as seen by the network
- Useful for comparing against device claims or detecting unexpected changes

**Use case:** Verify that your device IMEI matches what you expect, especially after firmware updates or if suspicious activity is detected.

---

## Option 2: Show IMSI

**Command:** `AT+CIMI`

Displays the **International Mobile Subscriber Identity** — the unique identifier stored on your SIM card that ties you to your subscription and phone number.

```
[*] Sending AT command: AT+CIMI
234011234567890
```

**What it tells you:**
- Confirms the SIM is recognized
- Shows your subscription identity independent of the device
- Useful for verifying SIM identity across device swaps

**Use case:** Confirm SIM activation, verify which subscription is active, or monitor IMSI when testing SIM-swapping scenarios.

---

## Option 3: Show Signal Strength

**Command:** `AT+CSQ`

Queries the current signal quality in real-time. The response includes a signal strength value (0-31) and an error rate percentage, displayed as a progress bar:

```
[*] +CSQ: 18 | [████████░░░░░░░░░░░░] 58%
```

**What it tells you:**
- How strong your connection to the network is
- Signal bars correspond roughly to these values:
  - 0-5: Very weak (nearly unusable)
  - 6-10: Weak
  - 11-15: Moderate
  - 16-20: Good
  - 21-31: Excellent

**Use case:** Troubleshoot connectivity issues, find the best location for signal, or verify that your device is properly connecting to the network.

---

## Option 4: Real-Time Signal Monitor

Launches a continuous monitor that displays signal strength updating every second:

```
[*] Real-time signal monitor (CSQ)
[*] Port: /dev/ttyUSB0
[*] Press Ctrl+C to exit
[*] +CSQ: 22 | [██████████████████░░] 71%
```

The display updates live, allowing you to:
- Walk around and observe signal changes in different locations
- Identify dead zones or signal drop areas
- Detect when your device switches between base stations

**Use case:** Find optimal antenna placement, map signal coverage in your area, or troubleshoot intermittent connectivity by observing real-time signal fluctuations.

---

## Option 5: Show BTS Antennas

**Command:** `AT+COPS?`

Queries your modem for Base Transceiver Station (BTS) information — details about the cellular antenna your device is currently connected to.

```
[*] Querying BTS antennas...
[*] BTS information:
    Mode     : 0
    Format   : 2
    Operator : 20801
    Status   : 1
    Info     : https://mcc-mnc.com/
```

**What it tells you:**
- **Operator ID** (MCC-MNC format): Identifies the carrier and country
  - Format: `MCC` (3 digits, country) + `MNC` (2-3 digits, network operator)
  - Example: `20801` = Spain (208) + Vodafone (01)
- **Mode**: Registration state (0=manual, 1=automatic, 2=deregistered)
- **Status**: Current connection status

Use [mcc-mnc.com](https://mcc-mnc.com/) to decode the operator ID and understand which carrier you're connected to.

**Use case:** 
- Verify which network operator you're connected to
- Detect unexpected roaming or network switches
- Understand coverage in your region
- Monitor which BTS antenna you're using (for location tracking awareness)

---

## Option 6: Change IMEI (Advanced)

**⚠️ LEGAL WARNING ⚠️**

Changing your device's IMEI is **illegal in many jurisdictions**, including but not limited to:
- United States (federal crime)
- European Union (criminal offense in most member states)
- United Kingdom
- Australia
- Canada

Unauthorized IMEI modification can result in:
- Criminal prosecution
- Device seizure and network banning
- Civil liability
- Fines and imprisonment

Use this option at your own risk. The authors accept no responsibility for any misuse, damage, or consequences resulting from its use. With great power comes great responsibility.

### How It Works

If you choose this option, SirrAT will prompt you to enter a new 15-digit IMEI:

```
[?] Enter new IMEI (15 digits): 357687091234567
[*] Writing new IMEI...
[*] IMEI successfully changed.
```

This modifies the modem's firmware to report a different IMEI to the network on next registration.

### Legitimate Use Cases

- Testing burner device configurations in isolated lab environments
- Authorized penetration testing of carrier systems
- Forensic analysis on seized devices
- Firmware development and modem testing
- Privacy-related use cases, **only where IMEI modification is expressly permitted by applicable law and regulations**.

**Again: Do not use this for evading law enforcement, fraud, or illegal activities.** Possession of tools capable of IMEI modification without clear legal justification may itself be illegal in your jurisdiction.

---

## Option 7: Reboot Modem

**Commands:** `AT+CFUN=0` (power off) → `AT+CFUN=1` (power on)

Sends a power cycle sequence to the modem without restarting the entire device:

```
[*] Rebooting modem...
[*] Modem reboot sequence sent.
```

This resets the modem firmware state, clears any hung connections, and re-initializes registration with the network.

**Use case:**
- Clear stuck connections
- Force re-registration after network loss
- Reset modem state during troubleshooting
- Recover from firmware glitches

---

## Understanding the Three Identities

To use SirrAT effectively and protect your privacy, understand the distinction between device and subscriber identities.

### BTS (Base Transceiver Station)

A BTS is the cellular base station (antenna tower) your phone connects to.

The mobile network logs:

* Which BTS your device connects to
* At what time
* With which identifiers (IMEI and IMSI)

By correlating BTS connections over time, operators (and entities with access to the data) can infer location history and movement patterns with significant accuracy.

---

### IMEI – Device Identity

The IMEI (International Mobile Equipment Identity) is a unique identifier assigned to the hardware device.

Key properties:

* Visible to the operator every time the device registers on the network
* Independent of the SIM card used
* Used for:

  * Blacklisting stolen devices
  * Detecting fraud
  * Correlating usage across SIM changes

If the same IMEI appears with multiple SIMs, the network can infer that the same physical device is being reused.

---

### IMSI – Subscriber Identity

The IMSI (International Mobile Subscriber Identity) identifies the subscriber (you), not the device.

Key properties:

* Stored in the SIM (or eSIM)
* Tied to:

  * Phone number
  * Billing records
  * Legal identity (in many countries)
* Used by the network to authenticate and track the subscriber

If the same IMSI appears on different devices, the network still sees the same person.

---

## Why Changing Only SIM or Only Device Is Not Enough

In high-risk environments, network metadata analysis combines device identity, subscriber identity, and location.

### Changing SIM Only (New IMSI, Same IMEI)

* The operator sees the same device reconnecting with a new subscriber identity.
* Past logs may already associate that IMEI with a known person.
* Correlation by location and usage patterns is trivial.

Result: the new SIM can be linked back to the original user.

---

### Changing Device Only (New IMEI, Same IMSI)

* The operator clearly sees the same subscriber using a new device.
* From a legal and surveillance standpoint, nothing changes.
* Call patterns, contacts, and locations remain associated with the same person.

Result: the identity is preserved.

---

## Burner Devices and Metadata Hygiene

SirrAT enables precise visibility into your device and subscriber identities, but it does not automatically anonymize you. When combined with disciplined operational practices:

* A **burner device** (new IMEI)
* A **burner SIM** (new IMSI)
* **Controlled locations** (avoid patterns)
* **Limited contacts** (reduce correlation)
* **Frequent resets** (use Option 7 to cycle modem state)

It becomes possible to **minimize network correlation**, effectively achieving the goals of a burner phone setup.

The advantage: unlike blindly swapping hardware or SIMs, **SirrAT allows verification and monitoring** (Options 1-5), turning theory into a measurable, controllable practice.

---

## Best Practices When Using SirrAT

1. **Monitor regularly**: Use Option 4 (real-time signal monitor) to understand your signal environment
2. **Verify identities**: Confirm IMEI/IMSI regularly with Options 1-2 to detect unexpected changes
3. **Know your operator**: Use Option 5 to understand which network you're connected to
4. **Document BTS patterns**: Track which antennas you connect to; correlations reveal location
5. **Reboot when needed**: Use Option 7 to clear modem state if experiencing issues
6. **Avoid IMEI changes in production**: Reserve Option 6 for lab environments and research only

---

## Summary

SirrAT on Sirr OS for PinePhone Pro is more than a debugging tool—it is a **practical enabler for understanding mobile network metadata**:

* Provides direct insight into IMEI, IMSI, and BTS exposure.
* Helps confirm that device/SIM swaps are effective.
* Supports disciplined metadata hygiene in high-risk scenarios.
* Bridges the gap between theory (burner phones) and operational reality.
* Enables real-time monitoring of signal quality and network registration.

Privacy in mobile networks is not achieved through a single technical trick, but through informed choices, legal awareness, operational discipline, and tools that provide visibility.

**Use SirrAT responsibly. Understand the legal implications in your jurisdiction. Respect network operators' terms of service.**

---

**Privacy is a right. Take it back — in your pocket.**
