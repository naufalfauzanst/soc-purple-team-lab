# LAB-006 evidence

Six operator-saved artifacts were collected from the local `soc-transfer-lab` folder and copied without byte changes. The analyst visually reviewed all four screenshots and parsed the saved JSON.

| Artifact | Provenance and scope |
|---|---|
| [Alert list](LAB-006-svchost-alert.png) | Historical dashboard rows; two svchost alerts at 03:40:15 on 3 October 2026; no unique alert ID visible |
| [Event body](LAB-006-svchost-event.json) | Operator-saved JSON body for PID 3168; no detection/routing envelope |
| [Upstream rule](LAB-006-svchost-rule.yaml) | Operator-saved third-party YAML; not a deployed-rule export |
| [Volume mapping](LAB-006-volume-mapping.png) | VM command output maps C: to HarddiskVolume3 |
| [Signature verification](LAB-006-signature-verification.png) | Current System32 file signature is valid; signer identity not shown |
| [Hash verification](LAB-006-hash-verification.png) | Current System32 file SHA-256 matches the event |

Dashboard times are recorded as displayed. Verification screenshots do not retain execution timestamps. The JSON parent's timestamp is not used as the event time. No credentials or installation keys were found in these selected artifacts. Raw telemetry and full endpoint state are not included.

## SHA-256 manifest

Hashes establish integrity of these saved artifacts, not authenticity of the original cloud event or historical endpoint state.

| File | SHA-256 |
|---|---|
| LAB-006-hash-verification.png | 28ED6C7A0E60F666B7B83282F4E448ACA509E4B221BF62EF637DA00C62952F03 |
| LAB-006-signature-verification.png | AF571EC838DB20E8660AB05E3446957C680D4CDB587DE0B4A908AA3905C640FD |
| LAB-006-svchost-alert.png | C7F7254BF5B99DAED007046FC0155A758E7CDE93531C62FB298DC939BBA87994 |
| LAB-006-svchost-event.json | 624C9B008BB831B1747F778CD326FB0BB5C50D80FE34A8CB01770D3B7DB878A5 |
| LAB-006-svchost-rule.yaml | 0E898372DBE3BFC679194C1DE0344F93D19303E711ACFD5EA7C536D5F69B0A81 |
| LAB-006-volume-mapping.png | 4B4A8EC28B4E4707B5192F9FA7839D3695AF04B8BF344CE4A82712F21BD53464 |