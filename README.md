# SOC Purple Team Lab

Documented SOC L1 investigations of authorized Windows SSH/SFTP activity and LimaCharlie EDR process detection and controlled response. The exercise connects operator actions to authentication logs, network traffic, endpoint processes, a scheduled SIEM alert, and a case disposition.

**SSH/SFTP case:** [SOC-L1-001: SSH access and synthetic file transfer](cases/SOC-L1-001-ssh-file-transfer.md). Conducted on 2 October 2026 by Naufal.

**EDR case:** [SOC-L1-002: EDR process detection validation](cases/SOC-L1-002-edr-notepad.md). Two synthetic rule tests passed, and a live Notepad launch generated LAB-002.

**EDR response case:** [SOC-L1-003: Controlled process termination](cases/SOC-L1-003-edr-process-response.md). A manual response returned error 0 for the target, and local checks confirmed no Notepad processes remained at verification time.

**Network response case:** [SOC-L1-004: Controlled isolation and release](cases/SOC-L1-004-edr-network-isolation.md). Fixed-IP checks supported connection blocking during reported isolation and recovery after release.

**Existing-rule triage case:** [SOC-L1-005: Hostname alert investigation](cases/SOC-L1-005-hostname-alert-triage.md). The analyst reviewed a third-party rule and closed operator-confirmed lab activity without unnecessary containment.

## Observed results

| Evidence | Result | What it establishes |
|---|---|---|
| OpenSSH Operational output | Accepted login for `soc_remote_lab` | Successful authentication |
| Wireshark | TCP/SSH exchange between host and VM | Network connection; encrypted application payload |
| Sysmon Event IDs 1 and 3 in Splunk | `sshd.exe -> cmd.exe -> sftp-server.exe` and endpoint tuple | Process and connection context |
| SCP output and destination SHA-256 | 95-byte synthetic CSV; source/destination hashes match | Transfer completion and byte equality for compared artifacts |
| Scheduled LAB-001 alert | 20:53:01 UTC+07 alert with 20:43:12.421 process event | Scheduled process-start detection worked |
| Shutdown verification | `sshd` stopped; host TCP port check false | SSH was unavailable at verification time |

The operator used a dedicated non-administrator account and known credentials. This exercise demonstrates authorized access and transfer, not exploitation or confirmed malicious exfiltration. A process-start alert alone cannot establish which file moved.

## Environment

```text
Windows host 192.168.56.1
  | SSH/SFTP over VirtualBox host-only network
Windows VM SOC-ENDPOINT-01 192.168.56.101
  | Sysmon -> Splunk Universal Forwarder
Splunk Enterprise: windows_endpoint index
```

Wireshark ran on the host's Ethernet 3 interface. The VM also had NAT for installation. Historical VM timestamps used UTC+02; investigation times are normalized to UTC+07 or explicitly labeled UTC.

## Contents

- [Case and disposition](cases/SOC-L1-001-ssh-file-transfer.md): timeline, findings, uncertainty, response, handover.
- [Detection and settings](detections/LAB-001.md) and [SPL](detections/LAB-001.spl).
- [SSH/SFTP triage playbook](playbooks/ssh-sftp-triage.md).
- [Reproduction guide](lab/README.md), with separate VM and host steps.
- [Evidence provenance and SHA-256 manifest](evidence/README.md).
- [Reusable L1 ticket](templates/l1-ticket.md).

## EDR extension

The existing Windows VM was onboarded to LimaCharlie. The analyst inspected process/account/parent context, validated match and non-match fixtures, reviewed a live detection, and documented benign authorized activity. This case validates reporting, not file-read monitoring or containment.

- [LAB-002 rule documentation](detections/LAB-002.md) and [export with test fixtures](detections/LAB-002.yaml).
- [EDR process triage playbook](playbooks/edr-process-triage.md).
- [LimaCharlie lab guide](lab/edr-limacharlie.md).
- [LAB-002 evidence inventory](evidence/lab002/README.md).

## Controlled response extension

LAB-003 validates manual termination of a designated Notepad process through EDR and a separate endpoint-state check. This is an authorized response test; no automatic alert-to-response chain or network isolation was validated.

- [Response exercise guide](lab/edr-process-response.md).
- [LAB-003 evidence](evidence/lab003/README.md).

## Network isolation extension

LAB-004 tested manual isolation and release on the owned VM. The operator reported True/False/True for a fixed TCP destination; retained evidence shows isolated failure and post-release recovery. Broad network coverage and full action audit metadata were not established.

- [Isolation guide](lab/edr-network-isolation.md).
- [LAB-004 evidence](evidence/lab004/README.md).

## Existing-rule investigation

LAB-005 demonstrates the distinction between a matched behavior and malicious intent. It attributes the existing Sigma-derived rule, records account/parent context and operator confirmation, and documents a benign-authorized disposition. No custom LAB-005 rule or suppression was created.

- [Rule review and attribution](detections/LAB-005-rule-review.md).
- [LAB-005 evidence](evidence/lab005/README.md).

## SOC L1 skills demonstrated

The case demonstrates alert review, authentication/process/network correlation, timezone handling, evidence integrity checks, an authorization-based disposition, escalation criteria, and a written handover. Screenshots and transcripts distinguish observed facts from assumptions. Raw captures stay local and are excluded from Git.

## Remaining work

- Investigate the difference between Splunk `_time` and embedded Sysmon `UtcTime` for Event ID 3.
- Forward OpenSSH Operational logs to the SIEM and validate parsing.
- Test benign and suspicious contexts and detection gaps; validate suppression across overlapping schedules.
- Add Wazuh or Microsoft Sentinel integration and extend response verification with action audits and scoped network coverage. Wazuh/Sentinel integrations are not implemented; manual isolation/release of a tested connection is documented in SOC-L1-004. LimaCharlie process telemetry and report-only detection are validated in SOC-L1-002; manual process termination is validated in SOC-L1-003. Sysmon is telemetry, not an EDR product.

Related work: [Splunk detection lab](https://github.com/naufalfauzanst/soc-splunk-detection-lab) and [phishing investigation lab](https://github.com/naufalfauzanst/soc-phishing-investigation-lab).


## False-positive investigation

[SOC-L1-006: Svchost masquerading alert triage](cases/SOC-L1-006-svchost-false-positive.md) documents a device-path mismatch in an existing rule. Volume mapping, a valid current-file signature, and a matching SHA-256 support a false-positive disposition for the investigated event. No suppression or rule modification was applied.

- [Existing-rule review](detections/LAB-006-rule-review.md).
- [LAB-006 evidence and integrity manifest](evidence/lab006/README.md).

## Network analysis baseline

[SOC-L1-007: DNS, TCP, and TLS baseline analysis](cases/SOC-L1-007-network-baseline.md) documents DNS answers, a TCP reachability test, and a separate HTTPS/TLS session. An offline review verified 20 packets in the selected TLS capture; the raw capture remains local. Host-side NAT limits VM attribution. No IDS deployment or alert was validated.

- [Network baseline guide](lab/network-baseline.md).
- [LAB-007 evidence and integrity manifest](evidence/lab007/README.md).