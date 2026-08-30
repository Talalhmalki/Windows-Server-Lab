# 07 - Dynamic Host Configuration Protocol (DHCP)

## Purpose

This phase deploys centralized IPv4 configuration for the isolated `virexon.local` lab. It covers DHCP authorization, scope design, DNS-related scope options, lease verification, an exclusion test, and a fixed client reservation.

## Verified environment

| Component | Configuration |
| --- | --- |
| DHCP server | `PC26.virexon.local` |
| Server address | `192.168.1.2/24` |
| Server operating system | Windows Server 2025 Standard Evaluation |
| Client | `PC-IT-01` running Windows 11 |
| Network | `192.168.1.0/24` |
| Subnet mask | `255.255.255.0` |
| Final scope | `192.168.1.26–192.168.1.245` |
| Lease duration | 8 days |
| Exclusion | `192.168.1.180–192.168.1.190` |
| Reservation | `PC-IT-01` → `192.168.1.50` |

## Addressing design

| Range | Intended use |
| --- | --- |
| `192.168.1.1–192.168.1.25` | Static infrastructure |
| `192.168.1.26–192.168.1.245` | DHCP scope |
| `192.168.1.180–192.168.1.190` | Addresses excluded from dynamic allocation |
| `192.168.1.246–192.168.1.254` | Outside the scope; retained for controlled infrastructure use |

The exclusion is inside the configured scope, while the upper infrastructure range is outside it. The reserved address `192.168.1.50` remains inside the scope and is bound to the client's MAC address.

## Implemented configuration

### Server authorization and scope

The DHCP role was installed on `PC26` and authorized in Active Directory. The final IPv4 scope starts at `192.168.1.26`, ends at `192.168.1.245`, and uses an eight-day lease.

### Scope options

| Option | Value | Purpose |
| --- | --- | --- |
| 006 - DNS Servers | `192.168.1.2` | Directs clients to the domain DNS server |
| 015 - DNS Domain Name | `virexon.local` | Supplies the domain DNS suffix |
| 003 - Router | Not configured | The host-only lab has no router or default gateway |

Omitting option 003 is intentional for this isolated topology. A production subnet with routed connectivity would normally advertise its real gateway.

## Validation

### Initial dynamic lease

The client was configured to obtain IPv4 settings automatically. The evidence shows `PC-IT-01` receiving `192.168.1.26`, with both DHCP and DNS supplied by `192.168.1.2`. The same lease appears in the server console.

### Controlled exclusion test

To prove behavior rather than rely only on a configuration screen, the pool was temporarily narrowed to `192.168.1.180–192.168.1.191` while `192.168.1.180–192.168.1.190` remained excluded. After release and renewal, the client received `192.168.1.191`.

This was a temporary test condition. The scope was then restored to the final `192.168.1.26–192.168.1.245` range; the exclusion remained unchanged.

### Reservation

A reservation associated `PC-IT-01` with `192.168.1.50`. Following lease renewal, the client received the reserved address, and the server displayed it as `Reservation (active)`.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Post-Install Authorization](Screenshots/01-DHCP-Post-Install-Authorization.png) | The DHCP post-install configuration and authorization task completed. |
| 02 | [Scope Configuration](Screenshots/02-DHCP-Scope-Configuration.png) | Final pool, eight-day lease, and exclusion configuration. |
| 03 | [Scope Options](Screenshots/03-DHCP-Scope-Options.png) | Options 006 and 015 are configured; option 003 is absent. |
| 04 | [Client Lease Verification](Screenshots/04-DHCP-Client-Lease-Verification.png) | Client-side receipt of a dynamic address and domain DNS settings. |
| 05 | [Server Address Lease](Screenshots/05-DHCP-Server-Address-Lease.png) | The initial client lease is tracked by the server. |
| 06 | [Exclusion Test Configuration](Screenshots/06-DHCP-Exclusion-Range-Test.png) | Temporary `.180–.191` pool with `.180–.190` excluded. |
| 07 | [Exclusion Behavior](Screenshots/07-DHCP-Exclusion-Verification.png) | The renewed client lease skips the exclusion and receives `.191`. |
| 08 | [Reservation Configuration](Screenshots/08-DHCP-Reservation-Configuration.png) | Reservation name, `.50` address, and client identifier. |
| 09 | [Reservation Client Verification](Screenshots/09-DHCP-Reservation-Verification.png) | The client receives `192.168.1.50` after renewal. |
| 10 | [Reservation Server Verification](Screenshots/10-DHCP-Reservation-Server-Lease.png) | The `.50` lease appears as an active reservation. |

## Design boundaries

- The server is also the domain controller and DNS server because this is a consolidated learning lab.
- The scope intentionally has no router option and therefore does not provide Internet connectivity.
- The exclusion test records a temporary pool change; it is not the final scope configuration.
- The screenshots validate one client and one reservation, not capacity, failover, or production high availability.

## Outcome

DHCP is authorized and operational. The final scope, DNS options, eight-day lease, exclusion behavior, client lease, and `PC-IT-01` reservation are all supported by paired server-side and client-side evidence.
