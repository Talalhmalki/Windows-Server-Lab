# 01 - Server Baseline and Environment Verification

## Purpose

This phase records the stable server baseline used by the VIREXON lab: host identity, operating-system edition, network configuration, time zone, and domain sign-in.

## Evidence timing

> The screenshots were captured after the server had already joined `virexon.local` and several roles were present. They verify the resulting lab baseline; they do not represent a chronological pre-deployment sequence. AD DS deployment is documented separately in [Phase 02](../02-Active-Directory).

## Verified baseline

| Item | Observed configuration |
| --- | --- |
| Server name | `PC26` |
| Operating system | Windows Server 2025 Standard Evaluation |
| Domain | `virexon.local` |
| IPv4 address | `192.168.1.2/24` (static) |
| Default gateway | None, by design on the isolated host-only network |
| DNS client | Local loopback (`::1` and `127.0.0.1` in the captured output) |
| DHCP on server NIC | Disabled |
| Time zone | `(UTC+03:00) Kuwait, Riyadh` |
| Virtualization | VMware Workstation Pro |

Server Manager also shows AD DS, DHCP, DNS, File and Storage Services, and WDS in the captured state. Their presence is an observation from the final baseline, not evidence that those roles were installed during this phase.

## Validation evidence

| Check | Evidence | What it establishes |
| --- | --- | --- |
| Server Manager state | [01 - Server Manager](Screenshots/01-Server-Manager.png) | Server is reachable and infrastructure roles are visible in the captured state. |
| System identity | [02 - System Properties](Screenshots/02-System-Properties.png) | Hostname, domain, OS edition, and system resources. |
| Full interface output | [03 - `ipconfig /all`](Screenshots/03-IPConfig-All.png) | Static IPv4 configuration, DNS loopback entries, and disabled DHCP on the server NIC. |
| IPv4 properties | [04 - IPv4 Configuration](Screenshots/04-IPv4-Configuration.png) | `192.168.1.2/24`, no gateway, and local DNS configuration. |
| Time configuration | [05 - Time Zone](Screenshots/05-Time-Zone.png) | UTC+03 time-zone selection. |
| Domain authentication | [06 - Domain Login](Screenshots/06-Domain-Login.png) | Successful sign-in using `VIREXON\Administrator`. |

## Design note

The missing default gateway is intentional for this host-only exercise. It isolates the lab from external networks; it would not be an appropriate default for a production server that requires routed connectivity, updates, or external DNS resolution.

## Outcome

The captured baseline establishes a consistent server identity and static network configuration for the remaining phases. DNS service configuration and service-level validation are covered in [Phase 06](../06-DNS), rather than being claimed as complete here.
