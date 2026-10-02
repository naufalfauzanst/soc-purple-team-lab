# Evidence inventory and provenance

Evidence was collected in a user-directed lab on 2 October 2026. Screenshots are supplied by the operator, not raw SIEM event exports. Case log excerpts are transcribed from user-supplied output; they must not be treated as signed forensic exports.

| Artifact | Provenance |
|---|---|
| [01 SSH network](screenshots/01-ssh-network.png) | Supplied Wireshark screenshot, interactive SSH source port 64741 |
| [02 transfer network](screenshots/02-transfer-network.png) | Supplied Wireshark screenshot, transfer source port 61321 |
| [04 process chain](screenshots/04-splunk-sftp-process-chain.png) | Supplied Splunk table with cmd/SFTP process rows |
| [05 triggered alert](screenshots/05-lab001-triggered-alert.png) | Supplied final scheduled alert screenshot at 20:53:01 UTC+07 |
| [06 scheduled results](screenshots/06-lab001-scheduled-results.png) | Supplied scheduled results: one event at 20:43:12.421 |
| [07 alert settings](screenshots/07-lab001-alert-settings.png) | Supplied alert configuration screenshot |
| [Synthetic CSV](synthetic/customer-data-demo.csv) | Copied from host receiver; 95 bytes; hash independently checked |

The numbering retains the exercise's evidence labels; no artifact 03 is supplied here. Screenshots include lab identifiers and local UI paths. They were reviewed before publication; no account password or private key is included. The CSV records are synthetic.

Two PCAPNG files remain in `C:\Users\ASUS\Documents\soc-transfer-lab` and are intentionally excluded from this repository. Hashes are recorded in [local-capture-manifest.csv](local-capture-manifest.csv) for identification. A tshark TCP conversation summary of `lab001-alert-validation.pcapng` identified two SSH conversations: host ports 50908 and 55104 to VM port 22, lasting approximately 3.281 and 2.449 seconds. This is a metadata check, not decrypted content analysis or a filename attribution. No absolute-time correlation of these two conversations with the alert has been established. The older capture has not been independently reanalyzed.

[SHA256SUMS.txt](SHA256SUMS.txt) identifies published evidence files. Hashes provide integrity comparisons, not proof of provenance or authorization. Do not treat encrypted SSH packets as proof of a particular file's contents.
