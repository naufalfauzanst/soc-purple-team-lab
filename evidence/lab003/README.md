# LAB-003 Evidence Inventory

| Artifact | Provenance |
|---|---|
| processes-before-response.png | Operator-supplied Processes screenshot copied from the chat attachment; identifies PIDs and PPID |
| LAB-003-edr-kill-response.png | Operator-saved Timeline screenshot located in the local lab folder; error 0 for PID 6928 |
| LAB-003-kill-response.json | Operator-saved event-body excerpt located locally; lacks routing and timestamp fields |
| LAB-003-process-termination-verification.png | Operator-saved PowerShell screenshot located locally; two process checks return no rows |

The operator performed the response. The assistant inspected the retained files and JSON structure, but did not independently execute the EDR command or check the VM's live state. No screenshots were modified. These images contain lab usernames and paths; no installation key or public IP is visible in the reviewed artifacts.

The Timeline image supplies a dashboard display time; it does not establish the configured timezone. The PowerShell image does not include a verification timestamp. The JSON is an excerpt, not a full forensic export. No process-tree kill or persistence-prevention claim should be derived from the absence of the remaining Notepad process.

[SHA256SUMS.txt](SHA256SUMS.txt) identifies these files as prepared for the repository. Integrity hashes do not independently prove their origin or the operator's authority.
