# LAB-002 Evidence and Provenance

| Artifact | Source and interpretation |
|---|---|
| live-alert.png | Operator-saved dashboard screenshot; independently located in the local lab folder |
| rule-tests.png | Operator-supplied chat attachment showing 2 of 2 passing; copied without image edits |
| detection-transcript.redacted.json | Selected-field transcription of operator-supplied JSON; not an original export |
| ../../detections/LAB-002.yaml | Actual local platform rule export, including match/non-match fixtures |

The screenshot and export files were located locally. The original live detection JSON was reported as saved but was not found in the checked lab, Documents, or Downloads paths. The transcript preserves selected event fields and epoch timestamps while omitting author email, external IP, tenant/sensor/installer identifiers, and the source link. Hostname and username are retained as lab context. Original chat evidence is not reproduced wholesale.

The manual file-read confirmation came from the operator; no EDR file-read event or independent disk-content output was established. Historical onboarding errors are setup context rather than evidence of a failed live process detection.

[SHA256SUMS.txt](SHA256SUMS.txt) records the byte hashes of this evidence and the exported YAML as prepared for the repository. Hashes establish comparability, not authorization or authenticity. The repository remains private; privacy is not a substitute for protecting installation keys.
