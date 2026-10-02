# SOC-L1-001 — SSH Access and Synthetic File Transfer

## Ticket

| Field | Value |
|---|---|
| Analyst | Naufal |
| Date | 2026-10-02 |
| Status | Closed — investigation, scheduled alert, and SSH shutdown verified |
| Intake | Manual exercise review, followed by LAB-001 scheduled alert validation |
| Target | SOC-ENDPOINT-01, Windows VM, 192.168.56.101 |
| Operator / receiver | Windows host, 192.168.56.1 |
| Account | SOC-ENDPOINT-01\soc_remote_lab, dedicated non-administrator lab account |
| Disposition | Authorized remote access and transfer of synthetic data |
| Confirmed impact | A 95-byte synthetic CSV copied from the VM to the host |
| Final assessed severity | Informational for this authorized exercise |

## Ringkasan untuk L1

Windows utama mengakses Windows VM melalui SSH menggunakan akun latihan, lalu mengambil file pelanggan palsu melalui SCP. Log OpenSSH mencatat autentikasi, Wireshark memperlihatkan koneksi SSH, dan Sysmon di Splunk memperlihatkan proses SFTP. Hash file yang diterima sama dengan hash sumber yang dilaporkan pengguna.

Aktivitas dan transfer benar-benar terjadi, tetapi seluruh pengujian diizinkan oleh pemilik lingkungan. Tidak ada eksploitasi, password curian, malware, atau data pelanggan asli yang digunakan. Kasus ditutup sebagai aktivitas lab yang diizinkan. Alert LAB-001 kemudian berhasil diuji: proses SFTP pukul 20:43:12.421 tampil pada hasil jadwal 20:53, dan Triggered Alerts mencatat alert pukul 20:53:01 WIB.

## Investigation Timeline

Times below are normalized to UTC+07:00, except where explicitly labeled UTC. The VM originally used W. Europe Standard Time (UTC+02 on this date); the host used SE Asia Standard Time (UTC+07). The user changed the VM display timezone to SE Asia Standard Time after discovering the mismatch. This does not rewrite older logs or demonstrate clock synchronization.

| Time / evidence | Observation |
|---|---|
| 20:13:48, normalized from VM 15:13:48 | OpenSSH recorded a reset from host source port 63591; a preceding connection attempt failed before login |
| 20:15:26, normalized from VM 15:15:26 | Accepted password for soc_remote_lab from 192.168.56.1:64741 |
| Interactive session | Remote hostname and whoami returned SOC-ENDPOINT-01 and soc-endpoint-01\soc_remote_lab |
| Sysmon UtcTime 13:23:13.472 | Event ID 3 recorded host 192.168.56.1:61321 connecting to VM 192.168.56.101:22 through sshd.exe |
| 20:23:14, normalized from VM 15:23:14 | OpenSSH accepted password for soc_remote_lab from port 61321, then recorded disconnect |
| Sysmon UtcTime 13:23:14.264 | cmd.exe created with parent sshd.exe, running `/c "sftp-server.exe "` |
| Sysmon UtcTime 13:23:14.307 | sftp-server.exe created with parent cmd.exe under soc_remote_lab |
| SCP terminal output | customer-data-demo.csv transferred successfully: 100%, 95 bytes |
| Destination verification | Source and received-file SHA-256 matched |

The network event's Splunk `_time` displayed 20:23:08.661, while embedded `UtcTime` was 13:23:13.472 (20:23:13.472 UTC+07). This approximately five-second difference is separate from the resolved timezone display difference. Use embedded UTC, endpoint tuples, and process identifiers for correlation; timestamp parsing/source differences remain uninvestigated. Do not infer sub-second transfer duration or response metrics from this record.

## Evidence and Findings

### 1. Authentication

User-supplied OpenSSH Operational output:

```text
sshd: Accepted password for soc_remote_lab from 192.168.56.1 port 64741 ssh2
sshd: Accepted password for soc_remote_lab from 192.168.56.1 port 61321 ssh2
sshd: Disconnected from 192.168.56.1 port 61321
```

This identifies successful account authentication. It does not identify a transferred filename. The server ED25519 fingerprint was compared on the VM and host before the user accepted the SSH host key.

### 2. Network

The user supplied Wireshark screenshots for source ports 64741 and 61321. Both show TCP establishment, SSH protocol identification/key exchange, and encrypted packets between the host and VM. Filters used:

```text
ip.addr == 192.168.56.101 && tcp.port == 64741
ip.addr == 192.168.56.101 && tcp.port == 61321
```

Capture file: `ssh-file-transfer.pcapng` (retained locally, excluded from Git). File presence was checked; no independent packet-level reanalysis of the saved capture has yet been performed. Packet screenshots were reviewed during the exercise. The SSH capture does not reveal passwords, command contents, or the CSV contents in plaintext.

### 3. Endpoint process chain

[Splunk process evidence](../evidence/screenshots/04-splunk-sftp-process-chain.png) shows:

```text
sshd.exe -> cmd.exe -> sftp-server.exe
```

The cmd.exe ProcessGuid was `{3f37587f-b042-6abf-d704-000000001600}`. The sftp-server.exe ProcessGuid was `{3f37587f-b042-6abf-d904-000000001600}`. Both event rows contain the dedicated lab account. ParentImage relationships were observed; a full ParentProcessGuid join was not performed.

SCP invoked an SFTP subsystem in this exercise. This supports transfer-capable activity; process creation alone does not establish the filename, transfer direction, or whether a transfer completed.

### 4. Transfer confirmation

Source: `C:\SOC-Lab\customer-data-demo.csv` on the VM.

Destination: [published synthetic artifact](../evidence/synthetic/customer-data-demo.csv) on the host.

Size: 95 bytes. SHA-256, user-reported source and independently checked destination:

```text
674F87AF8EF39D6AE351FC2DF40CD3E7CD9AC39C123C2E49F9AA1D7BEA502D0B
```

The CSV contains only DEMO-001 / User Alpha and DEMO-002 / User Beta marked SYNTHETIC. Hash equality confirms byte equality of the compared artifacts; it does not make an unrelated file safe or prove authorization.

## Scheduled Detection Validation

The user created LAB-001 Windows SFTP Server Process Started, with Sysmon Event ID 1, exact sftp-server.exe path suffix, cron 3-59/5 * * * *, a 10-minute lookback, and Low digest severity. Alert settings were reviewed in the [settings screenshot](../evidence/screenshots/07-lab001-alert-settings.png).

A new SCP validation transfer completed with 95 bytes. The [scheduled results screenshot](../evidence/screenshots/06-lab001-scheduled-results.png) covers 20:43:00 through 20:53:00 and contains one sftp-server.exe event at 20:43:12.421 under soc_remote_lab. The [Triggered Alerts screenshot](../evidence/screenshots/05-lab001-triggered-alert.png) confirms the 20:53:01 scheduled Low digest alert. This validates the lab process-start detection; it does not prove malicious exfiltration or associate that event with a particular destination filename. A later final-named file exists locally, but a distinct later process event was not established by the supplied results.

An earlier result used a two-hour range and included older activity; it is not used as evidence for this final 10-minute validation. Overlapping search windows may alert on the same event more than once. Cross-run suppression and suspicious-versus-benign coverage tests remain unvalidated.

## Scope and Disposition

The observed scope is one VM, one host, one lab account, and one synthetic file. Authorization comes from the user-directed lab, not from the username, filename, or private IP alone. Broader account history, other endpoints, and unrelated transfers were not exhaustively searched.

Close the lab case without L2 escalation because the operator and purpose are known, the activity matches the authorized exercise, and the confirmed transferred data is synthetic. This is not a claim that an enterprise environment is uncompromised.

## Escalation Criteria for an Equivalent Real Alert

Escalate according to the organization's playbook if authorization cannot be confirmed, the account/source/destination is unexpected, sensitive data may have moved, privileged access is unexplained, or suspicious persistence/additional activity appears. Provide the user, device, UTC timeline, source/destination tuple, process chain, file evidence if available, scope, confidence, and unresolved questions.

SSH/SFTP traffic alone can be legitimate administration. Investigate context before calling it malicious exfiltration. Do not disable accounts or isolate production devices without the authority required by the response process.

## Response and Cleanup

Completed: captured traffic, reviewed logs and processes, checked destination hash, corrected the VM display timezone.

The user confirmed exiting the interactive SSH session and saving the validation capture as lab001-alert-validation.pcapng. The user then stopped sshd in the VM and supplied Get-Service output showing Stopped. A subsequent Test-NetConnection from the host to 192.168.56.101:22 returned TcpTestSucceeded: False. This supports closure of the SSH listening service at verification time; it does not prove device isolation or removal of all persistence. Exact cleanup timestamps were not supplied. The lab account and narrowly scoped firewall rules were retained unless separately removed; removal was not validated. No EDR isolation, Wazuh response, or account disablement was validated. The LAB-001 scheduled process-start alert was validated. Reviewed screenshots and the synthetic CSV are published here. Raw packet captures remain local and are excluded from Git; see the evidence inventory.

## Handover

One authorized SSH session and one synthetic-file transfer were confirmed. No compromise established. Follow-up work: resolve the Event ID 3 timestamp discrepancy, decide whether OpenSSH Operational logs should be forwarded to Splunk, extend the validated process-start detection with legitimate and suspicious test cases, and retain the documented SSH-stop verification and decide whether to remove the remaining lab account and firewall rules.
