# EDR Process Alert Triage for SOC L1

1. Capture detection name/ID, endpoint, event time, generation time, and rule conditions. Confirm whether the evidence is raw telemetry, a detection, a transcript, or a screenshot.
2. Review executable path, command line, account, process identifier, parent identifier, and available ancestry. Signing and known paths are context, not conclusive trust signals.
3. Verify authorization and expected behavior through the owner or approved work record. Do not infer permission from a lab-looking filename alone.
4. Correlate available network/file/authentication events. Keep process execution, requested file access, observed file operations, and completed transfer as separate claims.
5. Scope related activity according to the playbook. Record unavailable sources and unreviewed alerts explicitly.
6. Close verified authorized behavior with evidence, or escalate unexplained/suspicious activity with a concise handover. Separate detection correctness from incident severity.
7. Perform response only within assigned authority. Record action, time, verification, and reversal when applicable. Do not represent an untested isolate button as completed containment.

For LAB-002, the expected disposition is benign authorized activity after confirming the operator's exercise. No automatic containment is configured.
