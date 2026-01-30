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

---

## SirrAT Overview

SirrAT is a tool included in SirrOS designed to interact with cellular modems using AT commands.  
It acts as a user-friendly CLI for sending commands to a modem through a serial interface, relying internally on tools such as atinout.

SirrAT is primarily intended for diagnostics, inspection, and controlled configuration of modem parameters on Linux-based mobile devices such as the PinePhone Pro.

Typical use cases include:

- Querying modem status and network registration
- Inspecting identifiers exposed by the modem (IMEI, firmware version, etc.)
- Sending standard AT commands without manually handling serial devices
- Providing a safer abstraction than raw `echo > /dev/ttyUSB*`

SirrAT does not bypass the mobile network, the operator, or legal constraints. It simply allows direct interaction with the modem firmware.

---

## Running SirrAT

SirrAT can be launched either from the graphical menu or directly from the command line.

```sh
sirrat
````

If administrative privileges are required (for example, to access the modem device), SirrAT will request them through PolicyKit or sudo, depending on system configuration.

---

## SirrAT Command Categories

SirrAT groups its functionality into logical command sets.

### Device Information

These commands query read-only modem information:

* Modem manufacturer and model
* Firmware version
* Supported radio technologies
* Device identifiers exposed by the modem

Typical outputs include:

* IMEI (International Mobile Equipment Identity)
* Software revision
* Hardware revision

These commands are commonly used for diagnostics and verification after flashing firmware or debugging connectivity issues.

---

### Network Status

Network-related commands allow you to inspect:

* Registration status (registered, roaming, searching)
* Signal strength (RSSI, RSRP, RSRQ depending on modem)
* Current radio technology (2G / 3G / LTE)

This is useful when troubleshooting:

* No service situations
* Poor signal quality
* Unexpected roaming behavior

---

### SIM Information

SirrAT can query the SIM for information such as:

* SIM presence
* IMSI (International Mobile Subscriber Identity)
* Operator identifiers (MCC/MNC)

This information helps confirm whether the modem correctly detects the SIM and whether the subscription is active.

---

## Advanced / Expert Commands

SirrAT provides an **advanced mode** intended for expert users; this mode enables **IMEI** changes via modem commands.

---

## Mobile Network Basics: BTS, IMEI, and IMSI

To understand the privacy implications of modem identifiers, it is important to clarify three core concepts.

### BTS (Base Transceiver Station)

A BTS is the cellular base station (antenna) your phone connects to.

The mobile network logs:

* Which BTS your device connects to
* At what time
* With which identifiers

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

The IMSI (International Mobile Subscriber Identity) identifies the subscriber, not the device.

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

SirrAT does not automatically anonymize your device, but it enables precise control. When combined with:

* A burner device (new IMEI)
* A burner SIM (new IMSI)
* Disciplined operational habits (controlled locations, limited contacts)

It becomes possible to **minimize network correlation**, effectively achieving the goals of a burner phone setup.

The advantage: unlike blindly swapping hardware or SIMs, **SirrAT allows verification and monitoring**, turning theory into a measurable, controllable practice.

---

## Summary

SirrAT on SirrOS for PinePhone Pro is more than a debugging tool—it is a **practical enabler for secure cellular operations**:

* Provides insight into IMEI, IMSI, and BTS exposure.
* Helps confirm that device/SIM swaps are effective.
* Supports disciplined metadata hygiene in high-risk scenarios.
* Bridges the gap between theory (burner phones) and operational reality.

Privacy in mobile networks is not achieved through a single technical trick, but through informed choices, legal awareness, and operational discipline.

