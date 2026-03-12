---
layout: post
title: DNSCrypt and DNS Privacy
subtitle: How SirrOS protects your DNS queries with dnscrypt-proxy and AdGuard
header-img: img/in-post/2026-3-14/header-dnscrypt.jpg
header-style: text
catalog: true
tags:
  - DNSCrypt
  - Privacy
  - DNS
  - SirrOS

---

## What is DNS and why does it matter?

Every time your device accesses a domain — `example.com`, `api.github.com`, anything — it first needs to resolve that name to an IP address. That resolution is handled by **DNS** (Domain Name System): a distributed query system that acts as the "phone book" of the Internet.

The problem: by default, those queries travel over the network in **plain text**. Any entity between you and the DNS server — your ISP, a compromised router, an attacker on the same WiFi network — can:

* See exactly which domains you query
* Modify responses to redirect you to fake sites
* Build a complete log of your Internet activity

---

## DNSCrypt: encryption and authentication for DNS

**DNSCrypt** is a protocol that encrypts and authenticates DNS queries between your device and the remote resolver.

Concrete guarantees it provides:

* **Encryption**: queries are unreadable by third parties in transit
* **Authentication**: the remote resolver is cryptographically verified; DNS server impersonation is not possible
* **Declarative anonymity**: compatible resolvers can certify that they do not log user queries

DNSCrypt is not the only encrypted DNS protocol — DNS-over-HTTPS (DoH) and DNS-over-TLS (DoT) also exist — but it is particularly efficient and has mature implementations for use on embedded and mobile devices such as the PinePhone Pro.

---

## dnscrypt-proxy

`dnscrypt-proxy` is the reference client for DNSCrypt. It acts as a **local DNS proxy**: it listens on `127.0.0.1:53` and forwards queries to the remote resolver using the encrypted protocol.

Key features:

* Support for DNSCrypt, DoH, and ODoH (Oblivious DoH)
* Configurable server filtering by properties (no logs, DNSSEC, no filters)
* Automatic selection of the fastest available server
* Public resolver list maintained by the community

---

## The resolver in SirrOS: AdGuard DNS Unfiltered

In SirrOS, `dnscrypt-proxy` is configured to use `adguard-dns-unfiltered-doh` as the primary resolver.

```toml
server_names = ['adguard-dns-unfiltered-doh']
```

This choice is deliberate. AdGuard offers two variants of their public DNS:

| Variant | Description |
|---|---|
| `adguard-dns` | Blocks ads and trackers |
| `adguard-dns-unfiltered-doh` | No filters — pure resolution |

SirrOS uses the **unfiltered variant** to avoid interfering with legitimate domain resolution. Content filtering is a user decision, not an OS decision.

AdGuard DNS Unfiltered explicitly declares that it **does not log user queries** and supports DoH with server authentication.

---

## Base configuration in SirrOS

The SirrOS configuration file is located at `/etc/dnscrypt-proxy/dnscrypt-sirros.toml`. The most relevant parameters:

```toml
# Explicitly selected resolver
server_names = ['adguard-dns-unfiltered-doh']

# Listen only on IPv4 loopback
listen_addresses = ['127.0.0.1:53']

# Enabled protocols
ipv4_servers = true
ipv6_servers = false
dnscrypt_servers = true
doh_servers = true
odoh_servers = false

# Requirements for remote servers
require_nolog = true      # Only servers that do not log queries
require_nofilter = true   # Only servers without their own blocklists
require_dnssec = false    # DNSSEC not mandatory (AdGuard supports it anyway)

# Never force TCP (unnecessary unless routing through Tor)
force_tcp = false
```

The proxy listens on `127.0.0.1:53` — meaning system applications send their DNS queries to the device itself, and `dnscrypt-proxy` handles encrypting and forwarding them to the remote resolver.

---

## Replacing systemd-resolved

In a standard Linux installation with systemd, DNS resolution is managed by **`systemd-resolved`**: a service that listens on `127.0.0.53:53` and acts as a local stub resolver.

The issue is that `systemd-resolved` and `dnscrypt-proxy` cannot coexist on loopback port 53 without additional configuration, and `systemd-resolved` does not encrypt queries by default.

In SirrOS, **`systemd-resolved` is disabled** and fully replaced by `dnscrypt-proxy`.

### Steps applied in SirrOS

**1. Disable and stop systemd-resolved:**

```sh
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

**2. Remove the resolv.conf symlink managed by systemd:**

```sh
sudo rm /etc/resolv.conf
```

**3. Create a static resolv.conf pointing to the local proxy:**

```sh
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
```

From this point on, all system DNS queries go to `127.0.0.1:53`, where `dnscrypt-proxy` is listening.

**4. Protect the file from being overwritten:**

```sh
sudo chattr +i /etc/resolv.conf
```

The immutable attribute prevents NetworkManager or other services from overwriting the file during network changes.

**5. Enable dnscrypt-proxy as a system service:**

```sh
sudo systemctl enable dnscrypt-proxy
sudo systemctl start dnscrypt-proxy
```

### Verification

To confirm the system is resolving correctly through the encrypted proxy:

```sh
# Check that dnscrypt-proxy is active
systemctl status dnscrypt-proxy

# Verify the local resolver responds
dig example.com @127.0.0.1

# Confirm which remote server is being used
dnscrypt-proxy -resolve example.com
```

---

## Summary

In SirrOS, DNS privacy is not a configuration option — it is the default behavior of the system:

* `systemd-resolved` is disabled to prevent unencrypted DNS leaks
* `dnscrypt-proxy` occupies local port 53 and handles all resolution
* The remote resolver is AdGuard DNS Unfiltered: no filters, no declared logs, DoH
* The `resolv.conf` file is protected against automatic overwrites

The result: no DNS query leaves the device in plain text, regardless of which network it is connected to.

**Privacy is a right. Take it back — in your pocket.**




