# Network traffic captures

This folder holds two packet capture files recorded on the OpenWRT VM with `tcpdump` and analysed in Wireshark on the Windows host.

---

## http_capture.pcap

Interface: `br-mng`  
Capture filter: `port 80`  
Protocol: HTTP

This file records HTTP traffic between the Windows host (`192.168.56.1`) and the OpenWRT web server (`192.168.56.2`). The capture command used was:

```bash
tcpdump -i br-mng -w /tmp/http_capture.pcap port 80
```

While the capture ran we loaded `http://192.168.56.2` in the browser and refreshed the page several times. The file contains the full HTTP requests and responses for those page loads.

What is visible in this capture:
- HTTP `GET /` requests
- HTTP `200 OK` responses with the full HTML source
- Request headers (`User-Agent`, `Accept`, `Accept-Language`)
- Response headers (`Content-Type: text/html`, `Content-Length`)

Because HTTP is plaintext, following an HTTP stream in Wireshark shows the page content exactly as sent — including the business name and student details. This demonstrates why unencrypted HTTP is risky for sensitive data.

How to open: Wireshark → File → Open → select `http_capture.pcap` → filter `http`.

---

## ssh_capture.pcap

Interface: `br-mng`  
Capture filter: `port 2222`  
Protocol: SSHv2

This file records an SSH session between the Windows host and the OpenWRT router on port 2222 (we changed SSH from port 22 during hardening). The capture command used was:

```bash
tcpdump -i br-mng -w /tmp/ssh_capture.pcap port 2222
```

A PuTTY session connected using key-based authentication, a few commands (`ls`, `exit`) were run, then the session was closed and the capture stopped.

What is visible in this capture:
- The SSHv2 key exchange handshake (visible in plaintext)
- All subsequent packets are encrypted — commands and output are not readable
- Source and destination IPs and ports are visible

SSH protects session content with encryption; an interceptor can see a connection occurred but cannot read the commands or data.

How to open: Wireshark → File → Open → select `ssh_capture.pcap` → filter `ssh`.

---

## Comparison summary

| Feature | http_capture.pcap | ssh_capture.pcap |
|---------|-------------------|------------------|
| Protocol | HTTP | SSHv2 |
| Port | 80 | 2222 |
| Data visible to interceptor | Full HTML, headers, content | Only IPs and ports |
| Commands/content readable | Yes — plaintext | No — encrypted |
| Risk level | High | Low |
