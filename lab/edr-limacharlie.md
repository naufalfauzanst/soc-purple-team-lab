# LimaCharlie EDR Lab: Process Telemetry and Detection

## Deployment

Create a dedicated organization and installation key in the LimaCharlie console. Install the official Windows x64 EXE sensor inside the owned Windows VM with administrator privileges using the vendor's installation instructions. Keep installation keys out of tickets, screenshots, Git, and shared command output. Use the host browser for the dashboard if the VM browser is slow.

Sensor onboarding transmits endpoint telemetry to LimaCharlie cloud services. Review the selected data region and current billing settings. The organization in this exercise displayed system-extension sensors separately from the endpoint quota; this does not establish that every extension or add-on is free. EPP subscription errors appeared during setup; EPP activation was not validated or required for the demonstrated process rule.

Verify the correct hostname and the sensor's `Receiving` state before testing. This exercise used Windows VM `SOC-ENDPOINT-01`; Sysmon and Splunk telemetry were already present, but no LimaCharlie-to-Splunk integration was configured.

## Generate and inspect telemetry

Create a synthetic text file `C:\SOC-Lab\edr-demo.txt`. In VM PowerShell:

```powershell
Start-Process -FilePath "$env:WINDIR\System32\notepad.exe" -ArgumentList 'C:\SOC-Lab\edr-demo.txt'
```

In the sensor Timeline, search for `notepad.exe` and inspect `NEW_PROCESS`. Ensure the displayed date/range covers the new activity. Record process/account/parent/command-line context. Modern application launching may produce more than one Notepad process; inspect the actual relationships.

## Validate and create the rule

Open Automation > D&R Rules and create a draft. Use the detect/respond sections from [LAB-002.yaml](../detections/LAB-002.yaml). In Test Rule, evaluate the included synthetic match and non-match examples, retain them as tests, and confirm both pass. Save/create the enabled rule.

Launch the designated Notepad command again after rule creation. Inspect Detections for LAB-002, then verify the triggering event's endpoint, process, account, argument, and time. Synthetic tests and live detection are different validation stages.

## Document and retain

Use [the EDR triage playbook](../playbooks/edr-process-triage.md) and [L1 ticket template](../templates/l1-ticket.md). Export the rule, preserve screenshots, and retain raw detection JSON locally. Redact email, external IP, installation keys, and unnecessary identifiers before sharing evidence.

For a later response exercise, establish the permitted action and verification method separately. LAB-002 performs reporting only; no isolation or process termination is claimed.

Vendor reference: [Windows sensor installation](https://docs.limacharlie.io/2-sensors-deployment/endpoint-agent/windows/installation/).
