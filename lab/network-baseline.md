# LAB-007: Network baseline with Wireshark

Use only an owned VM and authorized network traffic. Start capture on the host Ethernet interface carrying internet traffic before each command. NAT may change visible source addresses; avoid attributing background host traffic to the VM.

## DNS

In VM PowerShell:

```powershell
nslookup example.com
```

Stop capture and filter `dns.qry.name == "example.com"`. Compare query and response transaction IDs, then expand the response's Answers. A records contain IPv4 addresses; AAAA records contain IPv6 addresses. Only the A-answer screenshot was retained for this case.

## TCP

Start a new capture and test an address returned by the current DNS response. The exercise used:

```powershell
Test-NetConnection 172.66.147.243 -Port 443 |
    Select-Object RemoteAddress, RemotePort, TcpTestSucceeded
```

After stopping capture, filter `ip.addr == 172.66.147.243 && tcp.port == 443`. Identify SYN, SYN/ACK, and ACK with the same endpoint/port tuple. A TCP reachability test does not by itself negotiate TLS or request a web page. DNS answers and destination availability may change; the historical address is not a permanent dependency.

## HTTPS and TLS

Start a new capture, then run in the VM:

```powershell
curl.exe --http1.1 -I https://example.com
```

Stop capture and filter:

```text
tls.handshake.type == 1 && tls.handshake.extensions_server_name == "example.com"
```

Select the matching Client Hello, choose Follow > TCP Stream, then close the stream window to inspect the filtered packet list. Inspect Server Hello, encrypted records, and connection closure. A TLS 1.3 Application Data label can include encrypted handshake content. Compare with terminal output rather than claiming the capture reveals HTTP headers.

Export only displayed packets using File > Export Specified Packets > Displayed. Keep the raw capture local. This exercise provides a baseline; IDS deployment and detection validation are separate work.

See the [case findings](../cases/SOC-L1-007-network-baseline.md).