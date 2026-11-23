# Wireshark Packet Analysis Report

**Analyst:** [Your Name]  
**Date:** [Date and timezone]  
**Wireshark version:** [Version]  
**Operating system:** [OS/version]  
**Interface captured:** [e.g. Wi-Fi, Ethernet, lo, lo0, Npcap Loopback Adapter]  
**Authorization/scope:** [Describe your own device or approved lab]  

## 1. Capture details

- Capture file: `captures/wireshark_capture.pcap`
- Start time:
- End time:
- Total duration (must be at least 120 seconds):
- Total packets:
- Capture environment: [local host / isolated VM / lab]

## 2. HTTP traffic

- Display filter: `http`
- Screenshot: `../screenshots/02-http-filter.png`
- Packet number:
- Source IP:Port:
- Destination IP:Port:
- HTTP method:
- Request URI/path:
- Host header:
- Other visible non-sensitive fields:

### Security observation

Explain what data was visible because this was unencrypted HTTP. Use only the harmless lab request. Do not include credentials, cookies, tokens, or personal information.

## 3. DNS traffic

- Display filter: `dns`
- Screenshot: `../screenshots/03-dns-filter.png`
- Query packet number:
- Query name:
- Query type:
- Response packet number:
- Response address/result:
- UDP or TCP port:
- Security observation:

If no DNS packets appear, state why (for example, cached lookup, encrypted DNS, or capture on the wrong interface) and repeat a controlled lookup while capturing on the correct interface.

## 4. TCP three-way handshake

- Display filter used: `tcp`
- TCP stream number:
- Client IP:Port:
- Server IP:Port:
- Screenshot: `../screenshots/04-tcp-handshake-annotated.png`

| Step | Packet number | Flags | Sequence number | Acknowledgment number | Explanation |
|---|---:|---|---:|---:|---|
| 1 | [#] | SYN | [value] | [value/blank] | Client initiates connection |
| 2 | [#] | SYN, ACK | [value] | [value] | Server acknowledges client SYN and sends its SYN |
| 3 | [#] | ACK | [value] | [value] | Client acknowledges server SYN |

Confirm all three packets belong to the same TCP connection. If Wireshark shows relative sequence numbers, say so.

## 5. Security findings

1. **Observation:** [What you saw]
   - **Risk:** [Why it matters]
   - **Recommendation:** [Specific mitigation]

2. **Observation:** [What you saw]
   - **Risk:** [Why it matters]
   - **Recommendation:** [Specific mitigation]

## 6. Conclusion

Summarize what the capture demonstrated about HTTP visibility, DNS resolution, TCP connection establishment, and the value of encryption.

## 7. Evidence inventory

- [ ] `captures/wireshark_capture.pcap`
- [ ] `screenshots/01-interface-and-capture-duration.png`
- [ ] `screenshots/02-http-filter.png`
- [ ] `screenshots/03-dns-filter.png`
- [ ] `screenshots/04-tcp-handshake-annotated.png`

**Privacy review:** Confirm that the capture and screenshots contain no passwords, cookies, tokens, private messages, or unrelated third-party traffic before uploading.
