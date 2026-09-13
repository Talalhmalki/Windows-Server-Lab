# 13 - Active Directory Sites and Services

## Purpose

This phase extends `virexon.local` from one Domain Controller to two. It deploys `PC27` as an additional writable Domain Controller, places both DCs in the `Riyadh-HQ` site, validates DNS and directory-service readiness, exercises replication in both directions, and tests client authentication through `PC27` during a controlled outage of `PC26`.

## Verified environment

| System | Address | Role in this phase |
| --- | --- | --- |
| `PC26.virexon.local` | `192.168.1.2/24` | Existing writable DC, DNS server, DHCP server, and final FSMO role holder |
| `PC27.virexon.local` | `192.168.1.3/24` | Additional writable DC, DNS server, and Global Catalog |
| `PC-IT-01` | `192.168.1.50/24` | Windows 11 domain client and RSAT workstation |

| Directory component | Verified value |
| --- | --- |
| Domain | `virexon.local` |
| NetBIOS name | `VIREXON` |
| Active Directory site | `Riyadh-HQ` |
| Site subnet | `192.168.1.0/24` |
| Network | VMware VMnet1 host-only |
| Evidence | 9 screenshots |

The Domain Controllers use static IPv4 addresses. The client normally receives its reserved address from DHCP on `PC26`.

## Site and Domain Controller design

The subnet `192.168.1.0/24` was associated with `Riyadh-HQ`, and both `PC26` and `PC27` were placed in that site. The mapping allows domain members to derive their Active Directory site from their IP subnet.

`PC27` was promoted into the existing domain with:

- a writable directory replica;
- the DNS Server role;
- Global Catalog enabled; and
- placement in `Riyadh-HQ`.

This is an intrasite, two-DC topology. It does not demonstrate intersite replication, site-link cost, or site-link scheduling.

## Replication topology and service readiness

Active Directory Sites and Services displayed an automatically generated KCC connection object between the Domain Controllers. That capture verifies the visible inbound replication topology; the separate replication tests provide the functional direction checks.

On `PC27`:

- `SYSVOL` and `NETLOGON` were published as shares;
- a DNS query sent explicitly to `192.168.1.3` resolved `PC26.virexon.local` to `192.168.1.2`; and
- the AD service-location query returned records for both `PC26` and `PC27`.

These results show that the additional DC could serve the retained directory-related DNS data and publish the expected domain shares.

## Bidirectional replication validation

Manual **Replicate Now** operations completed successfully in both directions:

| Source | Destination | Captured result |
| --- | --- | --- |
| `PC26` | `PC27` | Replication completed successfully |
| `PC27` | `PC26` | Replication completed successfully |

The screenshots record successful replication operations. They do not display the contents of a separate test object, so the claim is limited to the operation and direction shown.

## Client site detection

From `PC-IT-01`, the domain account `VIREXON\adm-sami.ahmed` ran Domain Controller discovery checks. The client reported `Riyadh-HQ` as its site and discovered `PC26` in the same site.

| Check | Captured value |
| --- | --- |
| Client hostname | `PC-IT-01` |
| Client address | `192.168.1.50` |
| Client site | `Riyadh-HQ` |
| Discovered DC | `PC26.virexon.local` |
| DC address | `192.168.1.2` |

## Authentication failover test

`PC26` was deliberately shut down while `PC27` remained available. Because DHCP was hosted on `PC26`, `PC-IT-01` used a temporary static configuration during the outage test.

A fresh domain sign-in on the client produced:

| Verification | Captured result |
| --- | --- |
| Signed-in account | `VIREXON\adm-sami.ahmed` |
| Logon server | `\\PC27` |
| Discovered DC | `PC27.virexon.local` at `192.168.1.3` |
| Site | `Riyadh-HQ` |
| Secure channel | `NERR_Success` |

This validates domain authentication and the client secure channel through `PC27` during the controlled `PC26` outage. It does not demonstrate DHCP or file-service failover.

## Replication troubleshooting

During later validation, error `8524` reported that a DSA operation could not proceed because of a DNS lookup failure. A separate check also recorded a missing expected notification link. The rollout was treated as unhealthy until DNS registration/cache checks and directory synchronization had been completed and diagnostics were rerun.

The recovery sequence included:

```cmd
ipconfig /flushdns
ipconfig /registerdns
net stop dns && net start dns
repadmin /syncall /A /e /P /d
dcdiag /test:replications
```

The retained final `dcdiag /test:replications` capture on `PC26` shows both Connectivity and Replications passing, without the earlier notification-link warning. The evidence establishes recovery after the sequence; it does not isolate one command as the sole cause or prove an unobserved root cause.

## Evidence index

| # | Evidence | What it proves |
| ---: | --- | --- |
| 01 | [Site and Subnet Configuration](Screenshots/01-AD-Site-and-Subnet-Configuration.png) | `Riyadh-HQ` is associated with `192.168.1.0/24` and contains `PC26`. |
| 02 | [Additional Domain Controller](Screenshots/02-Additional-Domain-Controller-Deployment-Verification.png) | `PC26` and `PC27` are present in the site with Global Catalog enabled. |
| 03 | [KCC Connection](Screenshots/03-KCC-Replication-Connection-Verification.png) | An automatically generated replication connection is visible. |
| 04 | [DNS, SYSVOL, and NETLOGON](Screenshots/04-DNS-SYSVOL-NETLOGON-Replication-Verification.png) | `PC27` publishes the domain shares and answers the captured AD DNS queries. |
| 05 | [PC26 to PC27 Replication](Screenshots/05-PC26-to-PC27-Directory-Replication-Verification.png) | A manual replication operation from `PC26` to `PC27` succeeds. |
| 06 | [PC27 to PC26 Replication](Screenshots/06-PC27-to-PC26-Directory-Replication-Verification.png) | A manual replication operation from `PC27` to `PC26` succeeds. |
| 07 | [Client Site Detection](Screenshots/07-Client-Site-Detection-Verification.png) | The client and discovered DC report `Riyadh-HQ`. |
| 08 | [Authentication Failover](Screenshots/08-Secondary-DC-Authentication-Failover-Verification.png) | The client logs on through `PC27` and verifies its secure channel. |
| 09 | [Final Replication Diagnostic](Screenshots/09-Final-AD-Replication-Health-Verification.png) | Final Connectivity and Replications diagnostics pass on `PC26`. |

## Design boundaries

- Both DCs reside in one site and on one VMware host.
- The test demonstrates a Domain Controller VM outage, not physical-host resilience.
- DHCP and departmental file services remain on `PC26`.
- The client used temporary static addressing only because DHCP was unavailable with `PC26` offline.
- A passing replication diagnostic is reported exactly as captured; no broader high-availability claim is made.

## Outcome

`PC27` is operational as an additional writable Domain Controller, DNS server, and Global Catalog in `Riyadh-HQ`. The evidence confirms site mapping, replication operations in both directions, client site detection, authentication through `PC27` during a controlled outage, and a passing final replication diagnostic after troubleshooting.

**13 - Active Directory Sites and Services — Completed ✅**
