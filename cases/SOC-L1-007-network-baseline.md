# SOC-L1-007: DNS, TCP, and TLS baseline analysis

This authorized exercise examines ordinary DNS resolution and outbound HTTPS connectivity with Wireshark. It establishes a network-analysis baseline for later IDS work. No IDS was installed, no IDS rule was tested, and no malicious activity or security alert was demonstrated.

## Scope and vantage point

The operator ran commands in the owned Windows VM `SOC-ENDPOINT-01` and captured packets on the host's internet-connected Ethernet interface. The visible source is `192.168.1.31`; this is the host-side capture view, not a retained VM address. VirtualBox NAT and host background traffic limit attribution. Matching command target, SNI, and user-reported capture sequence support correlation; no pre-NAT capture or process/socket mapping was retained to independently prove VM origin.

## Findings

| Stage | Evidence | Result and limit |
|---|---|---|
| DNS | Answers screenshot | `example.com` returned A records `104.20.23.154` and `172.66.147.243`; resolution alone does not prove connection success |
| TCP reachability | Screenshot of fixed-IP test stream | SYN, SYN/ACK, ACK for `192.168.1.31:51919` to `172.66.147.243:443`, followed by closing packets; no application exchange proved |
| HTTPS | Operator-supplied curl transcript | `curl.exe --http1.1 -I https://example.com` returned `HTTP/1.1 200 OK`; headers were received, not the page body |
| TLS | Selected-stream screenshots and local capture | `192.168.1.31:60000` to `104.20.23.154:443`; Client Hello with SNI `example.com`, Server Hello, encrypted records, and TCP closure |

An initial exact-IP filter found no packets. A broader port filter exposed unrelated traffic, which was not attributed to the exercise. A repeat with a fixed destination produced the separate TCP-test screenshot. The TLS request subsequently used the other DNS answer. The TCP reachability and TLS screenshots are distinct connections and are not combined into one session.

## Capture verification

The operator exported the displayed TLS stream to a local PCAPNG file. An offline tshark inspection confirmed **20 packets**, all belonging to the one IPv4 endpoint pair and TCP port pair above. In the original screenshot, the stream is `229` and frame numbers start at `24918`; the exported subset renumbers frames from 1 to 20. The committed [packet summary](../evidence/lab007/LAB-007-packet-summary.tsv) uses the exported numbering:

- Frames 1–3: TCP establishment.
- Frame 4: Client Hello with `example.com` SNI.
- Frame 6: Server Hello and Change Cipher Spec.
- Later frames: records labeled Application Data and closing TCP packets.

The screenshot identifies TLSv1.3. TLS 1.3 records labeled Application Data may include encrypted handshake messages; they do not expose the HTTP request or response. No session secrets or decrypted payload were collected. The `200 OK` finding comes from curl output, not packet-payload inspection. SNI reveals a requested server name, not a URL path or proof of benign intent.

## Time and provenance

The exercise was performed on 4 October 2026. The transcript includes a server-supplied Date header `Sun, 04 Oct 2026 14:49:32 GMT`; this is not an independently verified endpoint execution timestamp. Screenshot times are relative capture times. No precise normalized investigation timeline is claimed.

## Disposition and next step

**Authorized network baseline; no incident established.** Wireshark supplied visibility and packet inspection, not an automated intrusion-detection decision. The next exercise should separately install or configure an IDS and verify a deliberately scoped lab rule against known traffic before claiming IDS detection capability. No blocking or containment occurred here.

See the [reproduction guide](../lab/network-baseline.md) and [evidence manifest](../evidence/lab007/README.md).