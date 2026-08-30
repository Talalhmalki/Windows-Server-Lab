# 06 - Domain Name System (DNS)

## Purpose

This phase implements and validates the internal DNS service required by the `virexon.local` Active Directory domain. The evidence covers forward and reverse resolution, Active Directory service discovery, secure dynamic updates, record aging and scavenging, client registration, and service health.

## Verified environment

| Component | Verified state |
| --- | --- |
| DNS server | `PC26.virexon.local` |
| Server address | `192.168.1.2/24` |
| Server operating system | Windows Server 2025 Standard Evaluation |
| Domain | `virexon.local` |
| Client used for testing | `PC-IT-01` |
| Client address during DNS testing | `192.168.1.31` |
| Client DNS server | `192.168.1.2` |
| Network design | VMware host-only; no default gateway |

The client address above reflects the DNS-test capture. A later DHCP phase assigns the same client a reserved address of `192.168.1.50`.

## Implemented DNS design

### Zones

| Zone | Type and purpose |
| --- | --- |
| `virexon.local` | Active Directory-integrated forward lookup zone |
| `_msdcs.virexon.local` | Active Directory service-location records |
| `1.168.192.in-addr.arpa` | Active Directory-integrated reverse lookup zone for `192.168.1.0/24` |

The final zone inventory also contains Windows-created reverse zones and `TrustAnchors`. Those built-in entries are retained but are not presented as custom lab deliverables.

### Host and pointer records

| Name or address | Record | Verified result |
| --- | --- | --- |
| `PC26` | A | `192.168.1.2` |
| `PC-IT-01` | A | `192.168.1.31` |
| `192.168.1.2` | PTR | `PC26.virexon.local` |
| `192.168.1.31` | PTR | `PC-IT-01.virexon.local` |

### Dynamic-update and record-lifecycle controls

| Control | Configuration |
| --- | --- |
| Dynamic updates | Secure only |
| No-refresh interval | 7 days |
| Refresh interval | 7 days |
| Server scavenging | Enabled |
| Scavenging period | 7 days |

Secure-only updates are appropriate because the primary zones are integrated with Active Directory. Aging and scavenging manage stale dynamic records; they do not remove a record merely because a DHCP lease expires.

## Functional validation

### Forward and reverse resolution

The server was queried explicitly to avoid ambiguity about which resolver answered:

```powershell
nslookup pc26.virexon.local 192.168.1.2
nslookup 192.168.1.2 192.168.1.2
nslookup 192.168.1.31 192.168.1.2
```

The captures show the expected `PC26` and `PC-IT-01` mappings. Reverse resolution of the client address was repeated from both the server and the Windows client.

### Client registration

The client registration workflow used:

```powershell
ipconfig /registerdns
ipconfig /flushdns
nslookup pc-it-01.virexon.local 192.168.1.2
```

The final lookup returned `192.168.1.31`. This proves that the client record was resolvable after registration; the screenshot is not a DNS audit log of the update transaction itself.

### Active Directory DNS health

```powershell
dcdiag /test:dns
```

The captured result shows `PC26` and `virexon.local` passing the DNS test.

### Zone, record, and service inspection

```powershell
Get-DnsServerZone
Get-DnsServerResourceRecord -ZoneName "virexon.local" -RRType A
Get-DnsServerResourceRecord -ZoneName "_msdcs.virexon.local" -RRType SRV
Get-Service -Name DNS
```

The evidence confirms the required zones, A records, Active Directory SRV records, and a running DNS Server service. The visible SRV entries include Kerberos, LDAP, and Global Catalog service locations pointing to `pc26.virexon.local`.

## Forwarder boundary

No external DNS forwarder is documented as part of the final configuration. The lab is intentionally isolated on a host-only network without a default gateway, so internal Active Directory DNS is validated independently of Internet name resolution. This is a lab constraint, not a production recommendation.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Reverse Lookup Zone Created](Screenshots/01-Reverse-Lookup-Zone-Created.png) | The `1.168.192.in-addr.arpa` reverse zone exists. |
| 02 | [PTR Record for PC26](Screenshots/02-PTR-Record-PC26.png) | `192.168.1.2` maps to `PC26.virexon.local`. |
| 03 | [Reverse DNS Resolution](Screenshots/03-Reverse-DNS-Resolution.png) | The server address resolves back to `PC26`. |
| 04 | [Forward DNS Resolution](Screenshots/04-Forward-DNS-Resolution.png) | `PC26.virexon.local` resolves to `192.168.1.2`. |
| 05 | [Client Forward Resolution](Screenshots/05-Client-Forward-DNS-Resolution.png) | The client resolves internal host records through `192.168.1.2`. |
| 06 | [DNS Diagnostic Test](Screenshots/06-DNS-Diagnostic-Test-dcdiag.png) | `dcdiag /test:dns` completes successfully. |
| 07 | [Aging Configuration](Screenshots/07-DNS-Aging-Scavenging-Configuration.png) | Seven-day no-refresh and refresh intervals. |
| 08 | [Server Scavenging](Screenshots/08-DNS-Server-Scavenging-Enabled.png) | Automatic scavenging is enabled with a seven-day period. |
| 09 | [Secure Dynamic Updates](Screenshots/09-Secure-Dynamic-Updates.png) | The forward zone accepts secure updates only. |
| 10 | [Client Registration Workflow](Screenshots/10-Dynamic-DNS-Client-Registration.png) | Registration commands and successful client-name resolution. |
| 11 | [Server-Side Client Reverse Lookup](Screenshots/11-Server-Reverse-DNS-Resolution.png) | `192.168.1.31` resolves from the server. |
| 12 | [Client-Side Reverse Lookup](Screenshots/12-Client-Reverse-DNS-Resolution.png) | The same reverse query succeeds from the client. |
| 13 | [DNS Zone Inventory](Screenshots/13-DNS-Zones-Configuration.png) | Final zone inventory and AD-integration flags. |
| 14 | [A Record Verification](Screenshots/14-DNS-A-Records-Verification.png) | Server, client, and AD-related A records. |
| 15 | [AD SRV Record Verification](Screenshots/15-AD-SRV-Records-Verification.png) | Service-location records used by Active Directory. |
| 16 | [DNS Service Status](Screenshots/16-DNS-Service-Status.png) | The DNS Server service is running. |

## Outcome

Internal DNS is operational for the `virexon.local` lab. Forward and reverse queries, secure client registration, Active Directory service discovery, record-lifecycle controls, diagnostic health, and service status are all supported by the captured evidence. The design remains intentionally limited to a single DNS/domain controller on an isolated lab network.
