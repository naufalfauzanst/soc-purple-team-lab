# SOC-L1-004: Controlled EDR Network Isolation and Release

| Field | Value |
|---|---|
| Analyst / date | Naufal / 2026-10-03 |
| Platform / endpoint | LimaCharlie / SOC-ENDPOINT-01 Windows VM |
| Exercise identifier | LAB-004 |
| Authority | Owner-directed lab response test |
| Intake | Controlled response exercise; no confirmed compromise or incident trigger |
| Disposition | Authorized isolation/release capability validation |
| Status | Closed for the tested connection; endpoint released according to operator report |

## Executive Summary

The operator isolated the Windows VM through LimaCharlie, checked outbound TCP connectivity, released isolation, and verified connectivity recovery. A hostname-based test initially resolved different addresses, prompting a second cycle against the fixed destination `172.66.147.243:443`.

The operator reported True before the fixed-IP isolation, False while isolated, and True after release. The retained fixed-IP screenshot shows the latter two results. This supports blocking and recovery of the tested connection; it does not establish that all network traffic was blocked.

## Preparation and Scope

The operator retained direct access to the VM desktop through VirtualBox. The EDR sensor was reported Receiving before the action. Isolation and release were performed manually on sensor `soc-endpoint-01`. The dashboard's Isolated and subsequent Allowed states were operator-reported; no status screenshot or full action audit record was retained in the supplied evidence.

The activity tested a benign outbound TCP connection. It was not a malware-containment test, and no automatic alert-to-isolation workflow was configured.

## Observations

### Initial hostname-based cycle

| State | Destination / result |
|---|---|
| Before isolation | `example.com` resolved to `172.66.147.243`; TCP 443 True |
| During isolation | Output table selected `104.20.23.154`; TCP 443 False |
| During isolation, detailed output | Failed TCP attempts were reported for both `104.20.23.154` and `172.66.147.243` |
| After release | Operator-supplied output showed `104.20.23.154:443` True |

The domain resolved multiple addresses. The changing address did not invalidate every comparison: the detailed output supported failure of the original address during isolation and recovery of the other address after release. A fixed-address test was nevertheless used to reduce ambiguity.

### Fixed-IP cycle

Command executed on the VM:

```powershell
Test-NetConnection 172.66.147.243 -Port 443 |
    Select-Object RemoteAddress, RemotePort, TcpTestSucceeded
```

| State | Result | Evidence source |
|---|---|---|
| Before second isolation | True | Operator report; not visible in the retained fixed-IP screenshot |
| During second isolation | False | Operator report and retained screenshot |
| After second release | True | Operator report and retained screenshot |

The earlier baseline screenshot independently shows successful connectivity to the same IP before the initial cycle. It is not labeled as a screenshot of the second cycle's baseline. No precise action times, response duration, or SLA metric can be established from these artifacts.

## Assessment

The same fixed destination failed during reported isolation and succeeded after reported release. Together with the supplied baseline, the observations are consistent with the intended containment behavior for that TCP connection.

The failed result alone would not prove EDR isolation: service availability, local networking, or other controls could also cause failure. The controlled action sequence and recovery strengthen the interpretation. An audit export and independent packet analysis would provide additional corroboration.

## Limits and Final State

- Scope: one owned Windows VM and TCP port 443 to the stated test destination.
- No broad protocol, inbound traffic, IPv6, host-only traffic, or alternate-destination coverage was tested.
- Continued cloud connectivity while isolated was not independently demonstrated by a retained heartbeat/audit artifact.
- No malware, exfiltration, or compromise was established.
- The operator reported release, and the final fixed-IP test succeeded. This proves recovery of the tested connection, not complete restoration of every service.
- The sensor and existing rules remain retained lab components unless separately removed; removal was not confirmed.

## L1 Handover

Close this controlled exercise with the verified connectivity results and stated evidence gaps. In an operational incident, network isolation requires the response authority prescribed by the organization's playbook and consideration of service impact. Preserve the incident basis, action target, UTC timestamps, acknowledgement, verification, and release authorization.

Follow-up: capture dashboard/action audit state, timestamp baseline and verification, and extend coverage only under an explicit test scope. Do not describe this case as comprehensive network-isolation assurance or NDR implementation.

## Evidence

- [Baseline screenshot](../evidence/lab004/LAB-004-network-before-isolation.png)
- [Initial isolated connection](../evidence/lab004/LAB-004-network-during-isolation.png)
- [Fixed-IP failure and recovery](../evidence/lab004/LAB-004-fixed-ip-validation.png)
- [Provenance and integrity manifest](../evidence/lab004/README.md)
- [Exercise guide](../lab/edr-network-isolation.md)
