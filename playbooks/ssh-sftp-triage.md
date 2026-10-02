# SSH/SFTP triage for SOC L1

1. Record alert name, run time, search window, device, user, event identifier, and evidence source. Verify telemetry freshness and distinguish event time, embedded UTC, and ingestion time.
2. Check whether SSH/SFTP is approved for this user, device, source, and time. Confirm with the asset owner or change record; private IPs and lab-looking filenames do not prove authorization.
3. Correlate authentication results, source/destination IP and port, process creation, parent relationships, and surrounding activity. Use ProcessGuid/ParentProcessGuid when available. Record correlation limitations.
4. Establish whether a file transfer actually occurred using available file auditing, transfer logs, or destination evidence. Encrypted traffic and `sftp-server.exe` alone are insufficient to identify a file. Document the source of every filename/hash claim.
5. Scope other activity by the account and endpoint, unexpected sources, privilege changes, persistence, and related alerts. Explicitly list sources or hosts not checked.
6. Disposition: close known authorized activity with supporting context, or escalate unexplained activity/sensitive transfer/suspicious behavior under the organization's playbook. Do not label all SFTP as exfiltration.
7. Provide L2 with an UTC timeline, account/asset, connection tuple, relevant events and process identifiers, file evidence, impact, confidence, and unanswered questions. Execute containment only within assigned authority; preserve evidence and verify its result.

Case closure should capture what was done, who authorized it, what verification showed, and remaining exposure. Stopping SSH is service shutdown, not EDR device isolation.
