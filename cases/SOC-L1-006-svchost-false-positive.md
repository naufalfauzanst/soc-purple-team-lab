# SOC-L1-006: Svchost masquerading alert triage

The analyst investigated an existing LimaCharlie alert on the owned Windows VM `SOC-ENDPOINT-01`. The retained evidence supports a false-positive disposition caused by comparing an NT device path with drive-letter paths. No process termination, isolation, suppression, or rule update was performed for this case.

## Alert and scope

- Alert: **Suspicious Process Masquerading As SvcHost.EXE**.
- Displayed alert time: **2026-10-03 03:40:15**, as shown in the dashboard; its timezone is not established by the saved event body.
- Investigated process: `svchost.exe`, PID **3168**, account `NT AUTHORITY\SYSTEM`.
- Parent: `services.exe`, PID **832**, account `NT AUTHORITY\SYSTEM`.
- Event path: `\Device\HarddiskVolume3\Windows\System32\svchost.exe`.

The alert screenshot contains two rows at the same displayed time and other historical detections. The saved event body was selected by the operator for this investigation; it lacks a detection ID, event type, routing envelope, and event timestamp. It cannot uniquely distinguish the two screenshot rows. The parent's `TIMESTAMP` is not treated as the alert timestamp.

## Evidence and findings

| Check | Observed result | Interpretation |
|---|---|---|
| Volume mapping | `fltmc volumes` maps `C:` to `\Device\HarddiskVolume3` | The device path resolves to the expected System32 location on this VM |
| Signature | `Get-AuthenticodeSignature` reports `Valid`, `Signature verified.` | The checked on-disk file has a valid signature; signer details were not retained |
| SHA-256 | `1222A44A5FDB4EFDE4DFCB41093648627950E7EC02D8667F1C26CCAE31D922E2` | The checked on-disk file's hash matches the event, ignoring letter case |
| Process context | Parent `services.exe`; both accounts `SYSTEM`; sensor reports signed files | Context supports ordinary service-host activity, but is not sufficient alone to prove safety |

The file and volume checks were performed after the alert. They support the investigation but do not independently establish the historical in-memory state or prove the endpoint is free of compromise. No network, service membership, memory, or wider endpoint investigation was completed in this case.

## Rule review

The operator supplied the upstream YAML for `windows_process_creation/proc_creation_win_svchost_masqueraded_execution`. It is attributed to **Swachchhanda Shrawan Poudel**, carries a high metadata severity, and reports the alert above. The analyst reviewed an existing third-party rule and did not author it.

The rule handles `NEW_PROCESS` and `EXISTING_PROCESS` on Windows. It requires a file path ending in `\svchost.exe`, then excludes exact case-insensitive matches to `C:\Windows\System32\svchost.exe`, `C:\Windows\SysWOW64\svchost.exe`, or an `ORIGINAL_FILE_NAME` of `svchost.exe`.

The saved event's device path satisfies the filename suffix but does not equal either drive-letter exception. `ORIGINAL_FILE_NAME` is absent from the saved body. This explains the suspected detection mismatch. The deployed rule was not exported and no engine replay was retained, so equivalence to the upstream YAML and missing-field evaluation were not independently tested.

## Disposition and handover

**Disposition: false positive supported by retained evidence; path-format mismatch identified as the likely trigger.** The mapped location, matching hash, valid signature, and parent/account context support this decision. The alert's high severity describes the rule's intended scenario, not a confirmed compromise in this event.

No containment was warranted by the evidence collected. For detection-engineering review, reproduce the event against the deployed rule and test a path-normalization fix that preserves detection of binaries outside approved locations. Include genuine System32/SysWOW64 paths, correctly mapped device paths, unexpected locations, and missing metadata. Do not broadly allow all device paths or disable the entire rule. No tuning proposal was applied or validated here.

Escalate if the mapping is inconsistent, signatures are invalid, hashes differ, the location is unexpected, or additional suspicious process/service/network evidence appears. Legitimate service-host files may still be abused; this finding closes only the investigated masquerading hypothesis.

## Evidence

See the [evidence inventory and SHA-256 manifest](../evidence/lab006/README.md) and [rule review](../detections/LAB-006-rule-review.md).