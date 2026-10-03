# LAB-003: Controlled Manual EDR Response

Purpose: learn how to identify a current process, execute an authorized endpoint action, and independently verify its outcome. The original test used a benign Notepad session, not malware.

1. Confirm the owned Windows VM is running and its LimaCharlie sensor is Receiving.
2. Open only the synthetic lab file in Notepad and save any intended edits before testing.
3. Identify current Notepad processes using VM PowerShell and the sensor's Processes view. Match the account, path, argument, and parent context. Never reuse a historical PID without checking the current process.
4. Select the intended test process. Record its PID and available process identity before using Kill Process. Ensure the permitted action and target are explicit.
5. Find the `OS_KILL_PROCESS_REP` reply and inspect its error code and target PID. Preserve the full envelope when possible, with sensitive metadata handled appropriately.
6. Check the target locally with `Get-Process -Id <current-target-pid>` and inspect any remaining relevant processes. Record the verification time and output.
7. Document action acknowledgement separately from the resulting state. If the response event is not initially visible, inspect the Timeline range and search without repeatedly issuing the command.

See [SOC-L1-003](../cases/SOC-L1-003-edr-process-response.md) for the observed result and limitations. This exercise does not validate automatic containment or network isolation. No LAB-003 detection rule was created.

Vendor reference: [LimaCharlie endpoint commands](https://docs.limacharlie.io/8-reference/endpoint-commands/).
