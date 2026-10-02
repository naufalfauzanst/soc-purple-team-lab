# Reproduce the authorized transfer exercise

Use only a VM and host you own. This guide uses a non-administrator lab account and synthetic data. Keep the host-only addresses below consistent with your environment. Take a VM snapshot before setup. The steps document a lab, not a production deployment.

## 1. Prepare the Windows VM

Configure VirtualBox host-only networking: VM `192.168.56.101`, host `192.168.56.1`. Keep NAT for downloads if needed. Install Sysmon and configure a Splunk Universal Forwarder to collect XML Sysmon Operational events into `windows_endpoint`; validate ingestion before testing. The exact Sysmon configuration and forwarder inputs were not exported in this case.

In **VM PowerShell as Administrator**, install/start OpenSSH and enable its operational log:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
wevtutil sl OpenSSH/Operational /e:true
```

Disable the installation's broad inbound rule if present, then create a rule limited to the host-only endpoint and host:

```powershell
Get-NetFirewallRule -Name OpenSSH-Server-In-TCP -ErrorAction SilentlyContinue | Disable-NetFirewallRule
New-NetFirewallRule -Name SOC-Lab-SSH-From-Host -DisplayName 'SOC Lab - SSH from Host' -Direction Inbound -Protocol TCP -LocalPort 22 -LocalAddress 192.168.56.101 -RemoteAddress 192.168.56.1 -Action Allow
$labPassword = Read-Host 'Password for dedicated lab account' -AsSecureString
New-LocalUser -Name soc_remote_lab -Password $labPassword -Description 'Authorized remote-access SOC lab'
New-Item -ItemType Directory -Path C:\SOC-Lab -Force
@'
customer_id,name,classification
DEMO-001,User Alpha,SYNTHETIC
DEMO-002,User Beta,SYNTHETIC
'@ | Set-Content -LiteralPath C:\SOC-Lab\customer-data-demo.csv -Encoding UTF8
Get-FileHash -LiteralPath C:\SOC-Lab\customer-data-demo.csv -Algorithm SHA256
ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub
```

Do not add the lab user to Administrators. Confirm the user can read the synthetic file. Encoding/newline behavior varies by PowerShell version, so compare your own source/destination hashes rather than expecting a fixed hash.

## 2. Capture on the Windows host

Select the host-only interface in Wireshark (Ethernet 3 in this case). Before starting, set **capture filter**:

```text
host 192.168.56.101 and tcp port 22
```

The main packet-window **display filter** has a different syntax:

```text
ip.addr == 192.168.56.101 && tcp.port == 22
```

Start capture. In a **new host PowerShell**, verify port 22 and connect:

```powershell
Test-NetConnection 192.168.56.101 -Port 22
ssh soc_remote_lab@192.168.56.101
```

Compare the presented host-key fingerprint with the VM before accepting it. Inside the remote shell, run `hostname`, then `whoami`, then `exit`. A password prompt uses the account's actual lab password; do not record it in evidence.

## 3. Transfer the synthetic CSV from VM to host

In **host PowerShell**, outside the SSH session:

```powershell
$receiveDir = Join-Path $HOME 'Documents\soc-transfer-lab\received'
New-Item -ItemType Directory -Path $receiveDir -Force
scp soc_remote_lab@192.168.56.101:C:/SOC-Lab/customer-data-demo.csv "$receiveDir\customer-data-demo.csv"
Get-FileHash -LiteralPath "$receiveDir\customer-data-demo.csv" -Algorithm SHA256
```

Compare against the VM hash. Stop capture and save all packets as a local `.pcapng`. Do not publish unrelated captured traffic.

## 4. Investigate and validate

Review OpenSSH Operational logs on the VM and Sysmon Event IDs 1/3 in Splunk. Load [LAB-001](../detections/LAB-001.spl), check results, and save using the [documented schedule](../detections/LAB-001.md). Perform another authorized synthetic transfer while capturing. Inspect the next scheduled alert's exact window and result event; an old event in a broad manual search is not validation of a new transfer. Record UTC timestamps, uncertainties, and disposition in the [ticket template](../templates/l1-ticket.md).

## 5. Shut down lab SSH

On **VM PowerShell as Administrator**:

```powershell
Stop-Service sshd
Get-Service sshd
```

On **host PowerShell**:

```powershell
Test-NetConnection 192.168.56.101 -Port 22
```

Expected: service Stopped and TCP check False. Retained accounts/firewall rules need a separate cleanup decision; this check does not establish device isolation. Capture and SIEM data should be preserved before cleanup.
