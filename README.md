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
| **Attack & Traffic Generation Tools** | `nmap`, `hping3`, `curl`, `wget`, `vsftpd` |
| **Network Setup** | Isolated virtual network with multi-node virtual machines (Target: `192.168.56.102` / `192.168.1.4`, Attacker/Host: `192.168.1.16` / `192.168.56.103`) |

---

### Scenarios & Findings
### Scenario 1: Real-Time Identification of Abnormal Network Behavior (Port Scan Detection)
* **Objective:** Detect an active reconnaissance attack against a target host live on the wire.
* **Methodology:**
1. Executed a TCP SYN port scan from host VM (`192.168.1.16`) targeted at target VM (`192.168.1.4`):     
```python
nmap -sS 192.168.1.4
```
2. Captured traffic in real time using the display filter:
```python
tcp.flags.syn == 1 && tcp.flags.ack == 0
```
3. Inspected traffic statistics via **Statistics > Conversations**.
* **Findings:** Captured a rapid burst of SYN packets targeting sequential ports on `192.168.1.4`. Identified the scanning host IP, port scan pattern, and confirmed open services before scan completion.
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_1.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_2.png)]

### Scenario 2: Gaining Visibility into Internal Network Traffic
* **Objective:** Capture internal node communications to identify unexpected protocols and unauthorized lateral traffic invisible to edge firewalls.
* **Methodology:** Set interface to promiscuous mode and monitored protocol distribution across all connected guest VMs.
* **Findings:**
  1. **Protocol Hierarchy Analysis:** Uncovered unexpected TCP/UDP protocol usage traversing the internal subnet.
  2. **Conversation Tracking:** Identified unauthorized session established between VM `192.168.1.16` and VM `192.168.1.4`, bypassing border security controls.     
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_3.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_4.png)]

### Scenario 3: Diagnosing Connectivity Issues (Network Exhaustion)
* **Objective:** Diagnose Layer 4 connection refusals and network resource exhaustion using Expert Info and I/O Graphs.
* **Methodology:**
1. Staged a connection exhaustion condition by launching a SYN flood targeted at port 80:
```python
sudo hping3 -S -p 80 --flood 192.168.56.102
```
2. Attempted HTTP requests using client curl requests
```python
curl http://192.168.56.102
```
3. Filtered connection reset flags:
```python
tcp.flags.reset == 1
```
* **Findings:**
1. **I/O Graphs** confirmed connection failures were driven directly by volume-induced queue saturation.
2. **Expert Information Engine** revealed TCP backlog saturation, showing the target server sending RST packets with window size $Win=0$ to drop incoming traffic.
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_5.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_6.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_7.png)]

### Scenario 4: In-Depth Incident Analysis (Plaintext Credentials & File Extraction)
* **Objective:**
1. Capture cleartext authentication sessions across insecure protocols (FTP & HTTP Basic Auth).
2. Perform post-incident packet forensics to reconstruct file exfiltration from raw `.pcap` files.
* **Methodology:**
1. **FTP Session:** Captured login attempt to target vsFTPd service. Used **Follow > TCP Stream**.
2. **HTTP Authorization:** Executed authenticated HTTP request:
```python
curl -u admin:password http://192.168.56.102/dvwa/login.php
```
3. **File Extraction:** Staged sensitive file retrieval via `wget http://192.168.56.102:8080/sensitive_data.txt`. Extracted file contents directly using **File > Export Objects > HTTP**.
* **Findings:**
1. Cleartext FTP credentials (`USER msfadmin`, `PASS msfadmin`) were fully exposed.
2. Base64-encoded HTTP Authorization header (Basic `YWRtaW46cGFzc3dvcmQ=`) was instantly decoded to `admin:password`.
3. The exfiltrated `sensitive_data.txt` file was successfully recovered bit-for-bit from packet payload.
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_8.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_9.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_10.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_11.png)]
[![Network-Traffic-Triage-Packet-Capture-Analysis ](assets/screenshot_12.png)]

---

### Key Wireshark Capabilities Used
| Feature | Key Application in Lab |
|---|---|
| **Deep Packet Inspection (DPI)** | Uncovering plaintext payloads & unauthorized protocols |
| **Follow Stream** | Reading cleartext FTP/HTTP transactions |
| **Protocol Hierarchy** | Discovering unexpected network traffic |
| **I/O Graphs** | Diagnosing denial-of-service/queue exhaustion |
| **Export Objects** | Forensic extraction of exfiltrated assets |
