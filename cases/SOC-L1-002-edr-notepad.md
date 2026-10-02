# SOC-L1-002: EDR Process Detection Validation

| Field | Value |
|---|---|
| Analyst / date | Naufal / 2026-10-02 |
| Platform | LimaCharlie |
| Endpoint | SOC-ENDPOINT-01, Windows VM |
| Detection | LAB-002 Notepad Demo Process Started |
| Disposition | Benign authorized activity |
| Status | Closed for this exercise |
| Response | Report-only detection; no containment performed |

## Executive Summary

An authorized lab exercise launched Windows Notepad with `C:\SOC-Lab\edr-demo.txt` as a command-line argument. A LimaCharlie endpoint sensor collected the process event, and the custom LAB-002 rule generated a live detection. The analyst reviewed the account, executable path, parent process, and command line and determined that the event matched the known exercise.

The detection establishes process creation with the designated file argument. It does not independently establish file access, successful reading, saving, or malicious behavior.

## Environment and Evidence Sources

The sensor was installed in the existing Windows VM, not the operator's primary Windows host. The operator supplied installation success output and a sensor Overview screenshot showing `Receiving`. The dashboard was accessed from the host. The sensor's internal IP was the VM's NAT address, `10.0.2.15`.

Evidence consists of operator-supplied dashboard screenshots, an independently located rule export, and a selected-field transcript of detection JSON supplied in chat. An original `LAB-002-live-detection.json` file was reported as saved by the operator but was not located in the checked Documents/Downloads paths. The transcript is explicitly labeled and must not be treated as a raw platform export.

## Timeline

| Time | Observation |
|---|---|
| Setup; exact time not independently established | Installer returned `Agent installed successfully`; dashboard showed the sensor receiving telemetry |
| 15:40:36-15:40:37, dashboard display | Early Notepad process events were observed |
| 15:48:06, dashboard display | A subsequent launch produced Notepad PID 7944 and child PID 9396 |
| Before live validation | Two synthetic tests passed: one match and one non-match; the exported rule contains both fixtures |
| 2026-10-02 16:05:30.691 UTC | Live `NEW_PROCESS` event, calculated from `routing.event_time=1790957130691` |
| 2026-10-02 16:05:32.233 UTC | Detection generation time, calculated from `gen_time=1790957132233` |
| 16:05:30, dashboard display | LAB-002 appeared in the sensor's detections list |

The live event corresponds to 23:05:30.691 at UTC+07. Epoch-derived times are UTC; the dashboard timezone configuration was not independently inspected. The difference between event and generation timestamps is approximately 1.542 seconds for this record; it is not an operational SLA measurement.

## Findings

### Process context

The live detection identified Notepad PID `6336`, running as `SOC-ENDPOINT-01\naufal`, with PowerShell PID `6204` as its parent. The executable path pointed to the Windows Notepad application directory. The command line was:

```text
"C:\WINDOWS\System32\notepad.exe" C:\SOC-Lab\edr-demo.txt
```

The sensor reported `FILE_IS_SIGNED=1` and executable SHA-256 `03745e9e21684c0e6d9fdc50d91caa0935f4baf50f626c11987ab7d6991d58d1`. Signing and a hash provide executable context; neither independently establishes benign intent. The parent relationship is reported within the event; no full ancestry investigation was performed.

### Detection validation

The rule requires a `NEW_PROCESS` event whose executable path ends with `\notepad.exe` and whose command line contains `C:\SOC-Lab\edr-demo.txt`, using case-insensitive comparisons.

| Fixture | Expected | Observed |
|---|---|---|
| Notepad with the designated file argument | Match | Match |
| Notepad without the designated file argument | No match | No match |

The dashboard screenshot shows `2 of 2 passing`. After the rule was created, the operator launched Notepad again and a live LAB-002 detection appeared. This establishes synthetic rule evaluation and a separate endpoint-to-detection validation.

### File-access limitations

The operator saved a training sentence and confirmed that it appeared after reopening the file. This is operator-reported manual confirmation of persistence and display. No file-read or write event was established from the EDR evidence, and terminal output independently verifying file contents was not supplied. The process command line alone cannot prove that Notepad successfully read the file.

## Disposition and Escalation

The reviewed account, process, argument, and operator-confirmed purpose were consistent with the authorized exercise. Classify the event as benign authorized activity rather than a demonstrated compromise. The rule detected its intended behavior, so benign context alone does not make the rule evaluation erroneous.

Escalate an equivalent unexplained event if the account or purpose cannot be verified, file sensitivity or related behavior raises concern, or broader investigation exceeds L1 authority. Provide the UTC timeline, process identifiers, executable and parent paths, account, command line, evidence sources, and unresolved questions.

This disposition applies to the reviewed event. Other detections previously visible in the organization were not investigated here, and no endpoint-wide assurance is claimed.

## Response, Handover, and Follow-up

The rule used a report action only. No process termination, network isolation, or automated remediation was performed. Sensor removal and rule disablement were not confirmed; the installed sensor and enabled rule should be treated as retained lab components.

Follow-up: test additional non-matching applications/arguments, inspect dashboard timezone settings, preserve a redacted original export if available, and conduct a separately authorized response exercise. Avoid claiming NDR integration or validated file-read telemetry from this case.

## Evidence

- [Live detection screenshot](../evidence/lab002/live-alert.png)
- [Two passing tests](../evidence/lab002/rule-tests.png)
- [Selected-field redacted transcript](../evidence/lab002/detection-transcript.redacted.json)
- [Exported rule and test fixtures](../detections/LAB-002.yaml)
- [Evidence inventory](../evidence/lab002/README.md)
