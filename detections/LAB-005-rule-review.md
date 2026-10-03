# LAB-005: Review of an Existing Hostname Rule

LAB-005 is the investigation identifier, not a newly created rule. The reviewed alert was `Suspicious Execution of Hostname` from a Sigma-derived rule supplied through LimaCharlie's Sigma extension.

Source: [refractionPOINT/sigma-limacharlie, rules branch](https://github.com/refractionPOINT/sigma-limacharlie/blob/rules/latest/windows_process_creation/proc_creation_win_hostname_execution.yml). Metadata in the operator-supplied source screenshot credits `frack113`; this repository does not claim rule authorship. Source context is retained in [the screenshot](../evidence/lab005/LAB-005-hostname-rule.png), rather than redistributed as an original custom rule.

Observed logic: Windows process telemetry (`NEW_PROCESS` or `EXISTING_PROCESS`), with a case-insensitive path suffix of `\HOSTNAME.EXE`, generates a Low report. Its discovery/T1082 tags classify the behavior; they do not prove an attacker performed it.

The rule is broad: legitimate troubleshooting can satisfy the same conditions. The analyst therefore reviewed account, parent, executable, command line, and operator authorization before assigning a disposition. No tuning or suppression was implemented. See [the case](../cases/SOC-L1-005-hostname-alert-triage.md).

The source review relies on the supplied screenshot and pasted rule contents. A separate automated fetch of GitHub failed; no current-content or deployed-version equivalence is claimed.
