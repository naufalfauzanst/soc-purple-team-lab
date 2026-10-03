# LAB-006: Existing svchost rule review

The [case investigation](../cases/SOC-L1-006-svchost-false-positive.md) reviews an existing Sigma-derived rule rather than introducing a custom detection.

- Rule identifier: `windows_process_creation/proc_creation_win_svchost_masqueraded_execution`.
- Metadata author: **Swachchhanda Shrawan Poudel**.
- Report name: **Suspicious Process Masquerading As SvcHost.EXE**.
- Metadata severity: **high**.
- Metadata technique: `attack.t1036.005`; tag `attack.stealth`, retained as supplied.
- [Operator-saved upstream YAML](../evidence/lab006/LAB-006-svchost-rule.yaml).

The rule's exact drive-letter exceptions do not match the event's NT device path. On the investigated VM, retained volume output maps that path to the expected System32 location. The current file's signature is valid and its SHA-256 matches the event.

Source provenance: the operator copied the YAML from the GitHub link exposed by the LimaCharlie rule page. This file is an upstream source copy, not a deployed-rule export. Its bytes are preserved, but source/deployed equivalence has not been independently verified. Original rule metadata and references remain in the YAML. No runtime rule changes or replay tests were performed.

Recommended follow-up is a detection-engineering review of path normalization with positive and negative test cases. Avoid a blanket exclusion for `\Device\HarddiskVolume*`: mapping and the remaining path matter.