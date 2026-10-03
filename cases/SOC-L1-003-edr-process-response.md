# SOC-L1-003: Controlled EDR Process Termination

| Field | Value |
|---|---|
| Analyst / date | Naufal / 2026-10-03 |
| Platform / endpoint | LimaCharlie / SOC-ENDPOINT-01 Windows VM |
| Exercise identifier | LAB-003 |
| Intake | Operator-directed response validation; no new incident trigger established |
| Target | Notepad.exe, PID 6928 |
| Authority | Owner-authorized lab exercise on a synthetic text-file session |
| Disposition | Benign authorized response test |
| Status | Closed for this exercise |

## Executive Summary

The operator used LimaCharlie's process controls to terminate a designated Notepad process on an owned Windows VM. The sensor returned `OS_KILL_PROCESS_REP` with `ERROR: 0` and `PROCESS_ID: 6928`. Subsequent local PowerShell checks returned no rows for either the target PID or any Notepad process.

The combined response and endpoint verification support successful termination of the designated lab process. This was a controlled response-capability test, not remediation of confirmed malware or a validated automated alert-to-response workflow.

## Preparation and Target Identification

The sensor was initially reported offline. The operator subsequently confirmed a `Receiving` state before proceeding. Local VM output identified two running Notepad processes:

| Process | PID | Start time, VM display |
|---|---|---|
| Notepad | 7944 | 2026-10-03 10:42:42 |
| Notepad | 6928 | 2026-10-03 10:42:43 |

The EDR Processes view showed PID 6928 with PPID 7944, both running as `SOC-ENDPOINT-01\naufal`. The parent process row contained the lab file argument. This ties the target to the current lab session. PID 7944 also appeared in an earlier exercise, so historical PID equality was not used as proof of process identity.

The operator was instructed to leave the synthetic-file session open without making new edits before the termination test. No claim is made about a complete inventory of unsaved application state.

## Action and Timeline

| Sequence / time | Observation |
|---|---|
| Before response | VM and EDR process listings identified target PID 6928 and parent PID 7944 |
| Operator action | Selected Kill Process from the target process menu |
| 2026-10-03 03:46:07, dashboard display | Timeline recorded `OS_KILL_PROCESS_REP` with error 0 for PID 6928 |
| After action; exact timestamp not captured | VM checks returned no process rows for PID 6928 or for process name Notepad |

The timestamp above is copied from the dashboard screenshot. The saved response object contains only `ERROR` and `PROCESS_ID`, not the routing envelope or an epoch timestamp. The dashboard timezone was not independently verified for this event; the response time is therefore not normalized to UTC. No response-duration or SLA metric is claimed.

The UI action initially appeared to lack a response event. A later Timeline search located the response; the kill command was not repeated solely to obtain evidence.

## Verification

### EDR acknowledgement

```json
{
  "ERROR": 0,
  "PROCESS_ID": 6928
}
```

The reply reports success for the expected target. The standalone JSON is an operator-saved event-body excerpt, not a full command/audit export. The Timeline screenshot provides event type and displayed time.

### Endpoint state

The operator ran:

```powershell
Get-Process -Id 6928 -ErrorAction SilentlyContinue |
    Select-Object ProcessName, Id

Get-Process -Name notepad -ErrorAction SilentlyContinue |
    Select-Object ProcessName, Id
```

The retained screenshot shows no process rows and a returned prompt. This supports that the target PID and other Notepad processes were absent at verification time. The absence of PID 7944 was also reported by the name-based check, but the available evidence does not establish exactly why that parent ended or prove a process-tree kill.

## L1 Assessment and Decision

The target and response matched the authorized test. The EDR reported a successful action, and a separate endpoint check supported the resulting state. Close the exercise as a verified manual process-termination test.

In operational SOC work, a benign Notepad launch alone would not justify termination. A response requires a documented incident basis or another approved purpose and the authority prescribed by the organization's playbook. Here the authority and purpose were the owner-directed exercise.

## Scope and Limitations

- One endpoint and one designated response target were tested.
- No new LAB-002 alert was supplied as the trigger for this action; the earlier detection case remains separate.
- No process-termination telemetry event, full command audit record, or full response routing envelope was retained.
- No network isolation, file deletion, account disablement, automated remediation, or endpoint-wide compromise assessment was performed.
- Verification establishes absence at one point in time, not prevention of future relaunch.
- Sensor shutdown/removal and rule disablement were not confirmed; retained lab components remain available for later exercises.

## Handover

Manual EDR process termination was validated on the controlled target. Further work: retain full response metadata and endpoint-verification timestamps, test permitted actions against documented scope, and establish reversal/verification procedures before a separate isolation exercise. Escalate operational cases if the response fails, the process returns unexpectedly, or scope/authority is unclear.

## Evidence

- [Process identification before response](../evidence/lab003/processes-before-response.png)
- [EDR command response](../evidence/lab003/LAB-003-edr-kill-response.png)
- [Saved response body](../evidence/lab003/LAB-003-kill-response.json)
- [Endpoint verification](../evidence/lab003/LAB-003-process-termination-verification.png)
- [Evidence provenance and manifest](../evidence/lab003/README.md)
