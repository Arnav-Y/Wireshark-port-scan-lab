# Wireshark Packet Analysis & Port Scan Detection Lab

A hands-on network security lab using Wireshark to capture, filter, and analyze live network traffic — culminating in running a self-built port scanner against a real target and identifying its packet-level signature.

---

## What this covers

- Live packet capture using Wireshark's interface, capture, and display layers
- Protocol filtering (DNS, TCP, HTTPS/TLS) to isolate specific traffic types
- TCP handshake and connection state analysis (SYN, SYN-ACK, ACK, FIN, RST)
- Generating real port scan traffic with a custom multithreaded Python scanner
- Identifying the packet-level signature of a port scan
- Full lab documentation with annotated screenshots

---

## Methodology

1. **Baseline capture** — captured live traffic on the Wi-Fi interface to observe normal background activity (MDNS broadcasts, DNS resolution, TLS sessions).
2. **Display filters** — isolated `dns`, `tcp`, and `https` traffic individually to study protocol behavior.
3. **TCP stream analysis** — tracked full connection lifecycles, including a normal HTTP redirect and an out-of-order SYN-ACK packet (identified as benign network jitter).
4. **Port scan generation** — ran a self-built multithreaded TCP port scanner ([`port_scanner_fast.py`](./port_scanner_fast.py)) against the network's default gateway while Wireshark captured traffic live.
5. **Evidence isolation** — applied the filter below to extract only the scanner's outbound connection attempts:

```
ip.dst==172.16.1.1 and tcp.flags.syn==1 and tcp.flags.ack==0
```

---

## Key finding

The scan produced **1024 SYN packets** from a single source IP to a single destination IP, covering nearly every port (1–1024), completed in **under 100 milliseconds**. This is the textbook signature intrusion detection systems use to flag port scanning activity:

- One source → one destination, many ports
- Extremely high packet rate (no human or normal app traffic behaves this way)
- SYN-only packets with no follow-up application data
- Ports arriving slightly out of order, consistent with concurrent multithreaded connections

### Important note on loopback traffic

An initial attempt to scan `127.0.0.1` (localhost) produced **zero visible packets** in Wireshark. This is expected — loopback connections are handled internally by the OS network stack and never reach the physical network adapter, so an interface-level capture can't see them. Scanning a real network host (the router) resolved this and is documented in the report.

---

## Files in this repo

| File | Description |
|---|---|
| `Wireshark_Port_Scan_Lab_Report.pdf` | Full lab report — methodology, filters, evidence, and analysis |
| `port_scanner_fast.py` | Multithreaded Python port scanner used to generate scan traffic |
| `port_scan_evidence.pcapng` | Raw Wireshark capture file containing the scan traffic |
| `screenshots/` | Annotated screenshots referenced in the report |

---

## Tools used

- **Wireshark** v4.6.6 — packet capture and analysis
- **Python 3** — `socket`, `threading` (port scanner)
- **Windows** — Wi-Fi interface capture

---

## Legal & Ethical Notice

> All scanning in this lab was performed against the author's own personal network and devices (own machine, own router) with full ownership and permission.  
> Scanning networks or systems you do not own or have explicit permission to test is illegal under the Computer Fraud and Abuse Act (CFAA) and equivalent laws worldwide.  
> This project is for **educational purposes only**.
