# Network Traffic Triage Packet Capture Analysis

### Overview
This project demonstrates the practical application of **Wireshark**—the industry-standard network protocol analyzer—to capture, decode, and analyze live network traffic within a controlled virtual lab environment.

---

### Lab Setup & Environment

| Component | Specification |
|---|---|
| **Operating System** | Kali Linux (2024.1+) |
| **Protocol Analyzer** | Wireshark 4.x |
| **Virtualization** | VMware Workstation / Oracle VirtualBox |
| **Attack & Traffic Generation Tools** | nmap, hping3, curl, wget, vsftpd |
| **Network Setup** | Isolated virtual network with multi-node virtual machines (Target: 192.168.56.102 / 192.168.1.4, Attacker/Host: 192.168.1.16 / 192.168.56.103) |

---

### Scenarios & Findings
### Scenario 1: Real-Time Identification of Abnormal Network Behavior (Port Scan Detection)
* **Objective:** Detect an active reconnaissance attack against a target host live on the wire.
* **Methodology:**
1. Executed a TCP SYN port scan from host VM (192.168.1.16) targeted at target VM (192.168.1.4):     
```python
nmap -sS 192.168.1.4
```
2. Captured traffic in real time using the display filter:
```python
tcp.flags.syn == 1 && tcp.flags.ack == 0
```
3. Inspected traffic statistics via **Statistics > Conversations**.
