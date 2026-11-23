# Task 6 – Capture and Analyze Network Traffic with Wireshark

## Objective

Capture at least two minutes of traffic on a network interface you own or are authorized to monitor. Use Wireshark display filters to inspect HTTP, DNS, and TCP packets; identify a TCP three-way handshake; and document security observations.

> **Important:** This repository does not contain a fabricated packet capture. You must create `captures/wireshark_capture.pcap` yourself by capturing traffic on your own device/lab. Never capture other people's traffic or publish captures containing credentials, session cookies, private messages, internal hostnames, or personal information.

## 1. Install Wireshark

Official download and installation instructions: https://www.wireshark.org/download.html

### Windows
1. Download the official Windows installer.
2. Run the installer and follow the setup wizard.
3. Install **Npcap** when the installer offers it. Npcap provides packet-capture support.
4. Start Wireshark. Capturing may require administrator privileges or the appropriate Npcap permissions.
5. Select the interface that is actually carrying your test traffic (for example, Wi-Fi or Ethernet).

### Ubuntu / Debian
```bash
sudo apt update
sudo apt install wireshark
```
If prompted whether non-superusers may capture packets, choose according to your system's security policy. On Debian/Ubuntu, add your account to the `wireshark` group only if your administrator approves:
```bash
sudo usermod -aG wireshark "$USER"
```
Log out and back in for group membership to take effect. Group membership grants packet-capture privileges; do not grant it casually. If you do not enable non-root capture, launch Wireshark with the approved elevated method for your distribution.

### macOS
1. Download the official macOS package from Wireshark.
2. Install Wireshark and follow the installer guidance for its packet-capture permissions/helper.
3. Approve required system permissions and select your active interface.

Verify the installation by opening Wireshark and checking that available interfaces show traffic counters changing.

## 2. Capture at least two minutes

1. Close unrelated applications and pause downloads, cloud sync, and messaging if practical. Use a test VM or isolated lab if possible.
2. Open Wireshark and choose your own active interface. Double-click it to start.
3. Capture for **at least 120 seconds**. During the capture, generate only controlled test traffic:
   - DNS lookup: `nslookup example.com` (Windows) or `dig example.com` / `nslookup example.com` (Linux/macOS).
   - TCP/HTTP test: follow the local HTTP demonstration below.
   - Normal local device traffic may also appear; do not inspect unrelated users' content.
4. Stop the capture with the red square button.
5. Save As `captures/wireshark_capture.pcap` (choose classic **pcap** format if prompted; `.pcapng` is also a valid Wireshark format, but this task specifically requests `.pcap`).

**Timing evidence:** Note the capture start/end times and ensure the elapsed time is at least 2 minutes. The capture file's packet timestamps provide additional evidence.

## 3. Generate a controlled plaintext HTTP request (recommended)

To demonstrate why HTTP is unsafe without exposing real credentials, create a local-only test server and request a harmless dummy path. This keeps the demonstration on your own computer.

Open a terminal in a working folder and run:
```bash
python -m http.server 8000 --bind 127.0.0.1
```
This serves the current directory only on the loopback interface. Keep it running.

In another terminal, run:
```bash
curl http://127.0.0.1:8000/demo?training=1
```
A 404 response is acceptable; the purpose is to generate a harmless HTTP GET request. Stop the server with Ctrl+C when finished.

Capture on the **loopback** interface:
- Windows: Npcap Loopback Adapter (if installed).
- Linux: `lo`.
- macOS: `lo0`.

If loopback capture is unavailable on your platform, use a private lab VM and a test web server that you control. Do not use real passwords, cookies, tokens, or private data. A local HTTP request demonstrates plaintext HTTP headers/path; it does not demonstrate interception of someone else's traffic.

## 4. Apply display filters

Enter these in Wireshark's **Display Filter** bar (not the Capture Filter bar):

| Purpose | Display filter |
|---|---|
| HTTP | `http` |
| DNS | `dns` |
| TCP | `tcp` |
| TCP SYN packets | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| TCP SYN-ACK packets | `tcp.flags.syn == 1 && tcp.flags.ack == 1` |
| TCP ACK packets | `tcp.flags.ack == 1 && tcp.flags.syn == 0` |
| One TCP stream | `tcp.stream eq 0` (replace `0` with the stream number you inspect) |
| Local test HTTP | `http && ip.addr == 127.0.0.1` |

If no HTTP packets appear on your normal network interface, that is expected on many modern networks because most web traffic uses HTTPS. Use the controlled localhost HTTP demonstration above and capture on loopback.

### HTTP analysis
Select the GET request and expand **Hypertext Transfer Protocol** in the packet details pane. For the local test request, document visible items such as:
- HTTP method (for example, `GET`)
- Request path (for example, `/demo?training=1`)
- Host header (for a local request, `127.0.0.1:8000`)
- Other headers that are present

Do not include any real secrets in screenshots or reports.

### DNS analysis
Select a DNS query and response. Record:
- Query name (domain)
- Query type (for example, A or AAAA)
- Response address, if present
- Whether the response was successful or returned an error

DNS is often visible in traditional UDP/TCP port 53 captures. DNS-over-HTTPS (DoH) and DNS-over-TLS (DoT) encrypt DNS traffic and will not appear as readable traditional DNS messages under the `dns` filter.

## 5. Analyze a TCP three-way handshake

Find a TCP connection in which you can see the initial connection establishment. A normal TCP handshake is:

1. **SYN** – client asks to start a connection; SYN flag is set and ACK flag is not set.
2. **SYN-ACK** – server acknowledges the client's SYN and sends its own SYN; both flags are set.
3. **ACK** – client acknowledges the server's SYN; ACK is set and SYN is not set.

Use the TCP flags filters above, then verify that all three packets belong to the **same connection** (same source/destination IP addresses and ports, and consistent sequence/acknowledgment numbers). Do not assume any three matching packets form one handshake.

In Wireshark, right-click a packet and choose **Follow → TCP Stream** when useful. Record the three packet numbers and annotate them in `reports/packet_analysis.md`.

## 6. Why unencrypted HTTP is dangerous and how HTTPS helps

HTTP sends application data without transport encryption. Anyone able to observe the network path may be able to read request paths, headers, and content. Depending on the application, that can expose credentials, session identifiers, personal information, and business data. An intermediary may also alter unprotected traffic.

HTTPS uses HTTP over TLS. When correctly configured and certificate validation succeeds, TLS provides encryption in transit, integrity protection, and authentication of the server's identity. This makes passive eavesdropping and undetected modification substantially harder. HTTPS does not protect a compromised endpoint, prevent phishing, or hide all metadata such as IP addresses and some connection information.

## 7. Save evidence

Add your own genuine files:
- `captures/wireshark_capture.pcap`
- `screenshots/01-interface-and-capture-duration.png`
- `screenshots/02-http-filter.png`
- `screenshots/03-dns-filter.png`
- `screenshots/04-tcp-handshake-annotated.png`
- Complete `reports/packet_analysis.md`

Before committing the pcap, inspect it for sensitive data. If it contains anything private, create a clean lab capture instead. GitHub repositories are often public.

## 8. Glossary

- **Packet:** A formatted unit of data sent across a network. It contains control information (such as addresses) and may carry part of an application's data.
- **Protocol:** An agreed set of rules that devices use to communicate, including message formats and expected behavior (for example, TCP, DNS, and HTTP).
- **Port:** A numbered logical endpoint used by a device to direct network traffic to the correct service or application (for example, TCP port 443 for HTTPS).
- **Payload:** The part of a packet that carries the actual data for the next protocol or application, rather than just delivery/control information.
- **Handshake:** A sequence of messages used by two devices to establish or negotiate a communication session. TCP uses SYN, SYN-ACK, and ACK to establish a connection.

## 9. References

1. Wireshark Foundation, **Wireshark User's Guide**: https://www.wireshark.org/docs/wsug_html_chunked/
2. Wireshark Foundation, **Display Filter Reference**: https://www.wireshark.org/docs/dfref/
3. Wireshark Foundation, **CaptureSetup**: https://wiki.wireshark.org/CaptureSetup
4. IETF, **RFC 9293 – Transmission Control Protocol (TCP)**: https://www.rfc-editor.org/rfc/rfc9293
5. IETF, **RFC 9110 – HTTP Semantics**: https://www.rfc-editor.org/rfc/rfc9110
6. IETF, **RFC 8446 – The Transport Layer Security (TLS) Protocol Version 1.3**: https://www.rfc-editor.org/rfc/rfc8446

## 10. Submission checklist

- [ ] Wireshark installed and installation/permissions documented.
- [ ] Captured at least 120 seconds on an authorized local interface.
- [ ] Saved `captures/wireshark_capture.pcap`.
- [ ] Captured and documented HTTP filter results.
- [ ] Captured and documented DNS filter results.
- [ ] Identified one complete TCP SYN → SYN-ACK → ACK handshake.
- [ ] Annotated packet numbers and sequence/acknowledgment details.
- [ ] Documented one harmless unencrypted HTTP request and its visible fields.
- [ ] Added genuine screenshots.
- [ ] Reviewed capture for private information before GitHub upload.

This is a hands-on task. The repository includes instructions and an analysis template, but you must perform the live capture and supply your own pcap and screenshots.
