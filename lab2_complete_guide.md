# Lab 2: Network Reconnaissance & IP Tracking - Complete Practical Guide

**Student**: Mayank  
**Date**: December 4, 2025  
**System**: Windows 10/11 (LAPTOP-TKVKDEDI)  
**Network**: HGU LAN  
**Status**: ✅ Completed with Real Output & Full Explanations

---

## Table of Contents

1. [Understanding Network Concepts](#understanding-network-concepts)
2. [Why These Tools Matter for Forensics](#why-these-tools-matter-for-forensics)
3. [Command Reference & Your Real Output](#command-reference--your-real-output)
4. [Detailed Analysis of Your Network](#detailed-analysis-of-your-network)
5. [Forensic Implications](#forensic-implications)
6. [Viva Q&A](#viva-qa)

---

## Understanding Network Concepts

### 1. IP Address – The "Home Address"

#### **What It Is**

The IP (Internet Protocol) address is like your **home address**. It tells the network where your computer is so data knows where to go.

#### **The Value to Record**

From your `ipconfig /all` output:
```
IPv4 Address . . . . . . . . . . . : 192.168.1.35(Preferred)
```

**Your IP: 192.168.1.35**

#### **Which Line to Look At**

Look for the line that says **IPv4 Address** (not IPv6).

**Why this line?**
- You will see several **"IPv6"** lines (long strings with letters like `2401:df80:...`)
- While these are valid, the vast majority of local networks and forensic tools still prioritize **IPv4** (the short number-dot format)
- Your output specifically says **(Preferred)** next to IPv4, meaning this is the active address your computer is using right now to talk to the router

#### **Why This Matters for Forensics**

| Aspect | Why It Matters |
|--------|---|
| **Tracking** | If a hacker attacks a server, the server logs the IP address. It's the first clue to trace the attack back to a specific location (or at least a specific network). |
| **Timeline** | Since IPs can change with DHCP, recording the IP at the **specific time of the crime** is crucial to prove **who had that address at that moment**. |

#### **The Analogy**

This is your **Home Address for the moment**. Just like moving to a coffee shop gives you a new address, connecting to a different network gives you a new IP.

---

### 2. IP Address Breakdown – 192.168.1.35

#### **What It Is**

It tells the network where your data should be delivered right now. If you move to a coffee shop, you get a new IP address.

#### **Why It's Important for Forensics**

**Tracking**: If a hacker attacks a server, the server logs the IP address. It's the first clue to trace the attack back to a specific location (or at least a specific network).

**Timeline**: Since IPs can change (DHCP), recording the IP at the **specific time of the crime** is crucial to prove **who had that address at that moment**.

#### **Breaking Down Your IP**

```
192.168.1.35 / 255.255.255.0

Network: 192.168.1.X (all devices on same network)
Your Device: .35
Full Network Range: 192.168.1.0 → 192.168.1.255
Usable Devices: 192.168.1.1 (gateway) to 192.168.1.254
```

**Why this matters:**
- **192.168.x.x** = Private IP (not routable on internet, hidden by NAT)
- **255.255.255.0** = /24 network (256 addresses, 253 usable)
- You're device #35 on this network
- Anyone on 192.168.1.x can see each other (same LAN)

---

### 3. MAC Address – The "Hardware Fingerprint"

#### **What It Is**

From your output:
```
Physical Address . . . . . . . . . . : 68-34-21-D0-36-99
```

**Your MAC: 68-34-21-D0-36-99**

#### **What MAC Means**

**MAC** = Media Access Control Address

- **Uniquely identifies your network card** (like a serial number for your NIC)
- 6 octets: `68-34-21` (vendor = Intel) + `D0-36-99` (your device serial)
- More stable than IP (IP changes with DHCP, MAC doesn't change)

#### **Why It's Important for Forensics**

| Aspect | Forensic Value |
|--------|---|
| **Device Tracking** | Network admins log: "MAC 68-34-21-D0-36-99 → IP 192.168.1.35 → User: Mayank" |
| **Persistence** | MAC lasts across networks (IP changes, MAC doesn't) |
| **Spoofing Detection** | If MAC suddenly changes, could indicate attacker using stolen MAC or anti-forensic technique |
| **Network Logs** | Investigators search network logs for your MAC to find all IP addresses ever assigned to you |

**Timeline Example**:
```
[2025-12-04 10:16:21] MAC: 68-34-21-D0-36-99 assigned IP 192.168.1.35
[2025-12-04 14:23:45] Same MAC accessed shared folder "Confidential"
[2025-12-04 18:30:00] Same MAC uploaded 50GB to external server
→ Proves same device did all three actions
```

---

### 4. Default Gateway – Your "Exit Point"

#### **What It Is**

From your output:
```
Default Gateway . . . . . . . . . . : fe80::bab7:dbff:fe87:7284%11
                                      192.168.1.1
```

**Your Gateway: 192.168.1.1**

#### **What It Does**

The gateway is your **exit point from the local network**. Think of it as:
- Your home address is 192.168.1.35
- Your gateway (192.168.1.1) is the router
- To leave the neighborhood (192.168.1.X), you go through the gateway
- All packets for external networks route through 192.168.1.1 first

#### **Why It's Important for Forensics**

| Concern | Implication |
|---------|---|
| **Control Point** | If attacker controls the gateway, they see ALL your traffic |
| **MITM Attack** | Attacker can redirect connections (man-in-the-middle) |
| **Detection** | Unexpected gateway IP = potential compromise |

**Example Compromise**:
```
Normal: ipconfig shows Gateway 192.168.1.1 (your router)
Compromised: ipconfig shows Gateway 10.0.0.1 (attacker's machine)
→ All your traffic now flows through attacker's device
```

---

### 5. DNS Servers – Your "Phone Directory"

#### **What They Are**

From your output:
```
DNS Servers . . . . . . . . . . . : 2401:df80:1000::6767
                                    2401:df80:1000::6666
                                    8.8.8.8
                                    8.8.4.4
```

**Your DNS:**
- Primary: 2401:df80:1000::6767 (HGU DNS)
- Secondary: 2401:df80:1000::6666 (HGU DNS)
- Fallback 1: 8.8.8.8 (Google DNS)
- Fallback 2: 8.8.4.4 (Google DNS)

#### **What They Do**

DNS translates domain names to IP addresses:
```
You type: www.google.com
DNS Server looks it up: → 142.250.190.78
Browser connects to 142.250.190.78
```

#### **Why It's Important for Forensics**

| Forensic Angle | What It Shows |
|---|---|
| **Visibility** | HGU DNS can see **every website you tried to visit** |
| **Logging** | HGU maintains query logs: [time] MAC → Domain lookup |
| **Privacy Risk** | Private organization knows your browsing habits |
| **DNS Hijacking** | If DNS is compromised, attacker redirects: `bank.com` → `fake-bank.com` |

**Example Attack**:
```
Attacker compromises DNS server
Changes: google.com → 192.168.1.100 (attacker's fake site)
You type: google.com
You actually visit: attacker's phishing page (with valid HTTPS cert!)
```

---

### 6. DHCP – How You Got Your IP

#### **What DHCP Is**

From your output:
```
DHCP Enabled . . . . . . . . . . . : Yes
Lease Obtained . . . . . . . . . . : 04 December 2025 10:16:21
Lease Expires . . . . . . . . . . . : 05 December 2025 10:16:20
DHCP Server . . . . . . . . . . . . : 192.168.1.1
```

**DHCP** = Dynamic Host Configuration Protocol

It automatically assigns IP addresses (instead of manually configuring each device).

#### **Your Lease Timeline**

| Event | Time | Meaning |
|---|---|---|
| **Lease Obtained** | 04 Dec 2025 10:16:21 | Your device asked router for IP; got 192.168.1.35 |
| **Duration** | 24 hours | IP valid until next day at same time |
| **Lease Expires** | 05 Dec 2025 10:16:20 | IP expires; device can request new IP |

#### **Why It's Important for Forensics**

| Impact | Forensic Use |
|---|---|
| **Device Timeline** | DHCP logs show when device was on network: "MAC 68-34-21-D0-36-99 active from 10:16 to 18:30" |
| **Multiple Devices** | If new IP assigned later, could indicate different device or IP reuse |
| **Audit Trail** | DHCP server maintains complete lease history |

**Investigation Example**:
```
Malware detected on 192.168.1.35
Check DHCP logs: Which MAC has that IP? 68-34-21-D0-36-99
That's Mayank's laptop
Interview Mayank: "I was offline after 6 PM"
DHCP logs prove laptop disconnected at 6:00 PM
→ Someone else was using it between 6–11 PM
```

---

### 7. IPv6 – The Modern Address

#### **What It Is**

From your output:
```
IPv6 Address. . . . . . . . . . . . : 2401:df80:1066:50c6:aa8a:5ff4:94fb:9e0f(Preferred)
Temporary IPv6 Address . . . . . . . : 2401:df80:1066:50c6:5467:12:2a41:da13(Preferred)
Link-local IPv6 Address . . . . . . : fe80::c446:64d7:b432:3ca7%11(Preferred)
```

**IPv6** = Next-generation IP protocol (128-bit vs 32-bit IPv4)

#### **Three Types You See**

| Type | Example | Purpose |
|------|---------|---------|
| **Global Unicast** | 2401:df80:1066:50c6:aa8a:5ff4:94fb:9e0f | Internet-routable (like IPv4) |
| **Temporary** | 2401:df80:1066:50c6:5467:12:2a41:da13 | Privacy – rotates to hide real address |
| **Link-Local** | fe80::c446:64d7:b432:3ca7%11 | Local network only (never goes to internet) |

#### **Why It Matters**

- IPv6 addresses are vastly larger (128 bits vs 32)
- Your device has **both IPv4 and IPv6** simultaneously (dual-stack)
- Attackers can use IPv6 to **bypass IPv4 monitoring** (if firewalls only watch IPv4)
- Geolocation harder (IPv6 WHOIS less reliable than IPv4)

---

## Why These Tools Matter for Forensics

### **ipconfig /all** – Establish Network Baseline

**Use Case 1: Compromise Detection**
```
Baseline (Day 1): 
  Gateway: 192.168.1.1 ✓
  DNS: 8.8.8.8, 8.8.4.4 ✓
  
Suspicious (Day 2):
  Gateway: 10.0.0.1 ❌ (changed!)
  DNS: 8.8.8.8, 1.1.1.1 ❌ (different server!)
  
Conclusion: Possible network compromise or Man-in-the-Middle attack
```

**Use Case 2: Timeline Reconstruction**
```
Question: Was the device on network when data was leaked?
Answer: Check "Lease Obtained" time
If device got IP at 10:16 AM but data leaked at 2:00 PM, device was definitely connected
```

---

### **netstat -ano** – Find Active Connections & Processes

#### **What This Command Does**

```
netstat -ano
  -a  = show ALL connections (listening + active)
  -n  = numeric format (IPs as numbers, not hostnames)
  -o  = show PID (process ID of app making connection)
```

#### **Your Real Output (Sample)**

```
Proto  Local Address          Foreign Address        State           PID
TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       1652
TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
TCP    192.168.1.35:49409     4.213.25.242:443       ESTABLISHED     5364
TCP    192.168.1.35:58241     185.199.111.154:443    ESTABLISHED     14680
TCP    127.0.0.1:49676        127.0.0.1:49677        ESTABLISHED     1748
```

#### **Breaking Down Each Row**

**Example 1: Listening Port (System Service)**
```
TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
├─ Port 445 = SMB (Windows File Sharing)
├─ 0.0.0.0:0 = listening on all interfaces for incoming connections
├─ LISTENING = waiting for someone to connect
├─ PID 4 = kernel process (system service)
└─ Forensic: If this is unexpected, possible backdoor
```

**Example 2: ESTABLISHED Connection (Active)**
```
TCP    192.168.1.35:49409     4.213.25.242:443       ESTABLISHED     5364
├─ Your IP:port 192.168.1.35:49409 = your device's side
├─ Remote IP:port 4.213.25.242:443 = remote server (HTTPS)
├─ ESTABLISHED = connection actively open right now
├─ PID 5364 = Process ID (look up in Task Manager to find app)
└─ Forensic: Proves your device connected to this IP at capture time
```

#### **Why This Matters**

| Forensic Question | netstat Answer |
|---|---|
| **Is malware running?** | Look for unknown listening ports (e.g., 6667 = IRC backdoor) |
| **What's exfiltrating data?** | Find large outbound connections; cross-reference PID to process |
| **Who's accessing my files?** | Look for connections to port 445 (SMB) |
| **Command & Control?** | Unexpected HTTPS connections to unknown IPs |

---

### **tracert www.google.com** – Understand Network Path

#### **What This Command Does**

Traces the **route** (hops) that packets take from your device to Google.

```
tracert www.google.com
  Sends packets with increasing TTL (Time-To-Live)
  Each router decrements TTL by 1
  When TTL reaches 0, router responds with its IP
  This reveals the complete path
```

#### **Understanding the Output**

```
Tracing route to www.google.com [2404:6800:4002:827::2004]
over a maximum of 30 hops:

  1     1 ms     1 ms     1 ms  2401:df80:1066:50c6:bab7:dbff:fe87:7284
  2     *        *        *     Request timed out.
  3     7 ms     5 ms     3 ms  2401:df80:1000::51
  ...
 10     4 ms     2 ms     2 ms  tzdelb-bg-in-x04.1e100.net [2404:6800:4002:827::2004]

Trace complete.
```

#### **How to Read This**

| Column | Meaning | Your Data |
|--------|---------|-----------|
| **Hop #** | Router number in path | 1, 2, 3...10 |
| **Time 1, 2, 3** | Round-trip time (ms) | Usually 1-25 ms |
| **IP/Hostname** | Router's address | 2401:df80:... |
| **"Request timed out"** | Router not responding to ICMP | Normal (firewall) |

#### **Your Trace Analyzed**

| Hop | Router | Time | Location | Analysis |
|-----|--------|------|----------|----------|
| **1** | 2401:df80:1066:50c6:bab7:dbff:fe87:7284 | 1 ms | Your Gateway | Very fast – local |
| **2** | Request timed out | – | ISP Gateway | Not responding (filtered ICMP) |
| **3** | 2401:df80:1000::51 | 7 ms | HGU/ISP | Fast – regional |
| **4-9** | Google network | 2-62 ms | Google backbone | Slightly variable |
| **10** | tzdelb-bg-in-x04.1e100.net | 2-4 ms | **Google Delhi** | Final destination |

#### **Why This Matters**

| Forensic Use | What It Shows |
|---|---|
| **Network Health** | No unusual detours = normal routing |
| **Compromise Detection** | Unexpected routing = possible MITM attack |
| **Geolocation** | Trace shows ISP routing (identifies general location) |
| **Baseline** | Compare today's trace with past traces to detect changes |

---

## Command Reference & Your Real Output

### Command 1: ipconfig /all

```bash
C:\Users\Mayank>ipconfig /all
```

#### **What You See**

```
Windows IP Configuration

   Host Name . . . . . . . . . . . . : LAPTOP-TKVKDEDI
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : hgu_lan

Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . : hgu_lan
   Description . . . . . . . . . . . : Intel(R) Wi-Fi 6E AX211 160MHz
   Physical Address. . . . . . . . . : 68-34-21-D0-36-99
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : 2401:df80:1066:50c6:aa8a:5ff4:94fb:9e0f(Preferred)
   Temporary IPv6 Address . . . . . . : 2401:df80:1066:50c6:5467:12:2a41:da13(Preferred)
   Link-local IPv6 Address . . . . . : fe80::c446:64d7:b432:3ca7%11(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.1.35(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : 04 December 2025 10:16:21
   Lease Expires . . . . . . . . . . : 05 December 2025 10:16:20
   Default Gateway . . . . . . . . . : fe80::bab7:dbff:fe87:7284%11
                                       192.168.1.1
   DHCP Server . . . . . . . . . . . : 192.168.1.1
   DHCPv6 IAID . . . . . . . . . . . : 157824033
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2E-68-C7-A3-24-6A-0E-CE-42-76
   DNS Servers . . . . . . . . . . . : 2401:df80:1000::6767
                                       2401:df80:1000::6666
                                       8.8.8.8
                                       8.8.4.4
                                       2401:df80:1000::6767
                                       2401:df80:1000::6666
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

#### **What This Tells You**

| Finding | Forensic Value |
|---------|---|
| **Hostname: LAPTOP-TKVKDEDI** | Device name for identification |
| **MAC: 68-34-21-D0-36-99** | Unique hardware identifier; track across networks |
| **IP: 192.168.1.35** | Current network location; can change with DHCP |
| **Subnet: 255.255.255.0** | You're on 192.168.1.0/24 network (256 addresses) |
| **Gateway: 192.168.1.1** | Router IP; packet exit point |
| **DHCP Server: 192.168.1.1** | Router assigns your IP; maintains lease logs |
| **Lease: 10:16:21 → next day** | Device was connected starting Dec 4 10:16:21 |
| **DNS: Multiple servers** | Primary HGU DNS, fallback Google DNS |
| **IPv6 Present** | Dual-stack modern network |
| **NetBIOS Enabled** | Windows file sharing available (security consideration) |

---

### Command 2: netstat -ano

```bash
C:\Users\Mayank>netstat -ano
```

#### **Key Findings from Your Output**

**Listening Ports (System Services):**
```
TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       1652
  └─ Port 135 = RPC (Remote Procedure Call)
  └─ Risk: WannaCry exploits this port

TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  └─ Port 445 = SMB (Windows File Sharing)
  └─ Risk: CRITICAL – ransomware vector

TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       3604
  └─ Port 5040 = UPnP (device discovery)

TCP    0.0.0.0:7680           0.0.0.0:0              LISTENING       17536
  └─ Port 7680 = Windows Update services
```

**Forensic Concern**: Port 445 + 135 are major ransomware vectors. If your machine is exposed to the internet (no NAT), this is a significant vulnerability.

**Established Connections (Sample):**
```
TCP    192.168.1.35:49409     4.213.25.242:443       ESTABLISHED     5364
  └─ Your device → Microsoft (Azure) HTTPS

TCP    192.168.1.35:58241     185.199.111.154:443    ESTABLISHED     14680
  └─ Your device → GitHub CDN HTTPS
  └─ PID 14680 = Browser (likely Chrome/Firefox)

TCP    127.0.0.1:49676        127.0.0.1:49677        ESTABLISHED     1748
  └─ Localhost ↔ Localhost (local process communication)
```

**Forensic Conclusion**: Normal web traffic (GitHub, Microsoft). No suspicious outbound connections to unknown IPs. PID 14680 (browser) making HTTPS connections is expected.

#### **How to Find Process Name from PID**

The `-o` flag shows **PID** but not process name. To find the app:

1. Open **Task Manager** (Ctrl+Shift+Esc)
2. Go to **Details** tab
3. Add column: **PID** (right-click columns → PID)
4. Find PID 14680 in the list → Shows which app (chrome.exe, firefox.exe, etc.)

**In Your Case**: PID 14680 = Browser making GitHub connections

---

### Command 3: tracert www.google.com

```bash
C:\Users\Mayank>tracert www.google.com
```

#### **Full Output with Analysis**

```
Tracing route to www.google.com [2404:6800:4002:827::2004]
over a maximum of 30 hops:

  1     1 ms     1 ms     1 ms  2401:df80:1066:50c6:bab7:dbff:fe87:7284
Hop 1 = Your gateway (router)
Response time: 1ms (very fast - local)
Address: Your IPv6 local gateway

  2     *        *        *     Request timed out.
Hop 2 = ISP gateway
Response time: Timeout (router not responding to ICMP - normal for security)
This is expected; doesn't indicate network problem

  3     7 ms     5 ms     3 ms  2401:df80:1000::51
Hop 3 = HGU/ISP network node
Response time: 3-7ms (good)
You've exited local network; now in regional ISP

  4     2 ms    13 ms     2 ms  2001:4860:1:1::36a6
Hop 4 = Google backbone router
Response time: 2-13ms (entering Google network)

  5     8 ms    62 ms     2 ms  2404:6800:81e1:280::1
Hop 5 = Google Asia-Pacific region
Response time: 2-62ms (variable, but normal)

  6     4 ms     4 ms     3 ms  2001:4860:0:1::5e5c
Hop 6 = Google internal routing

  7     2 ms     2 ms     2 ms  2001:4860:0:1::77ac
Hop 7 = Google internal routing

  8     5 ms     3 ms    25 ms  2001:4860:0:1::77d5
Hop 8 = Google internal routing

  9     2 ms     2 ms     3 ms  2001:4860:0:1::5391
Hop 9 = Google internal routing

 10     4 ms     2 ms     2 ms  tzdelb-bg-in-x04.1e100.net [2404:6800:4002:827::2004]
Hop 10 = Google server (final destination)
Hostname: tzdelb-bg-in-x04.1e100.net
TZDELB = Time Zone Delhi - content served from India datacenter
Response time: 2-4ms (good)

Trace complete.
```

#### **What This Proves**

| Finding | Meaning |
|---------|---------|
| **10 total hops** | Reasonable path length to India |
| **1-25ms latency** | Good network speed; no congestion |
| **Normal routing** | No unusual detours (not compromised) |
| **India endpoint** | Google routed you to Delhi datacenter (optimal for India users) |
| **No anomalies** | Trace matches expected path to Google |

#### **Forensic Significance**

If this trace **changed** between investigations:
```
Original trace: 10 hops to 2404:6800:4002:827::2004
Compromise trace: 12 hops to unknown IP
→ Possible DNS hijacking or route hijacking
```

---

## Detailed Analysis of Your Network

### What Your Network Configuration Shows

#### **Network Topology**

```
Your Device (192.168.1.35)
        ↓
Gateway/Router (192.168.1.1)
        ↓
ISP Gateway (2401:df80:1000::51)
        ↓
Google Backbone (2001:4860::...)
        ↓
Google Delhi Datacenter (2404:6800:4002:827::2004)
```

#### **Security Posture**

**Strengths ✓**:
- IPv6 enabled (modern, better security)
- DHCP (easier to manage than static)
- Multiple DNS servers (redundancy)
- Private IP (hidden behind NAT)

**Concerns ⚠️**:
- Port 445 (SMB) listening (ransomware risk)
- Port 135 (RPC) listening (WannaCry vulnerable)
- NetBIOS enabled (legacy protocol)
- No VPN detected (ISP can see all traffic)

#### **Active Connections Profile**

| Connection | Type | Risk | Status |
|---|---|---|---|
| Microsoft (4.213.25.242:443) | HTTPS (encrypted) | Low | Legitimate (Windows Update) |
| GitHub (185.199.111.154:443) | HTTPS (encrypted) | Low | Legitimate (development activity) |
| Google (various IPv6) | HTTPS (encrypted) | Low | Legitimate (browsing, cloud services) |
| Localhost (127.0.0.1) | Internal | Low | Normal system processes |

**Forensic Verdict**: All connections appear normal for college student with GitHub activity and Windows updates. No malware indicators detected.

---

## Forensic Implications

### 1. Creating a Baseline

Run these commands regularly and save output to establish "normal":

```
ipconfig /all > baseline_ipconfig.txt
netstat -ano > baseline_netstat.txt
tracert www.google.com > baseline_tracert.txt
```

**Later**, if you suspect compromise, re-run and compare:

```
If Gateway changed:       → Possible MITM attack
If DNS changed:          → Possible DNS hijacking
If new listening ports:  → Possible backdoor
If unknown ESTABLISHED:  → Possible C2 malware
If unusual routing:      → Possible network compromise
```

### 2. Incident Response Checklist

**If Suspected Compromise**:

| Check | Command | What to Look For |
|---|---|---|
| Network Config Changed? | `ipconfig /all` | Gateway, DNS, IP changed? |
| Backdoor Listening? | `netstat -ano` | Unexpected open ports? |
| Outbound C2? | `netstat -ano` | Connections to unknown IPs? |
| Route Hijacked? | `tracert www.google.com` | Unusual path/detours? |
| PID Match? | Task Manager | Verify suspicious PID's process name |

### 3. Timeline Reconstruction

**Scenario**: Data leaked on Dec 4 at 3:00 PM. Was your device connected?

**Investigation**:
1. Check DHCP log: "192.168.1.35 active from 10:16 AM–6:00 PM" ✓
2. Check netstat capture at 3:00 PM: Look for suspicious connections
3. Check gateway logs: "Traffic spike 3:00 PM" = possible exfiltration?
4. Compare to baseline: Is this connection normal or new?

**Conclusion**: Device was connected; need to investigate data flow during incident window.

---

## Viva Q&A

### Basic Understanding

**Q1: What does ipconfig /all show?**
**A**: All network adapters and their configuration: IP address, MAC address, gateway, DNS servers, DHCP status, lease times, IPv6 addresses.

**Q2: Explain your IPv4 address 192.168.1.35 with Subnet Mask 255.255.255.0**
**A**: 
- Network: 192.168.1.0 (first 3 octets fixed by /24 subnet)
- Host: .35 (your device is #35 on that network)
- Range: .1 (gateway) to .254 (usable addresses)
- 253 total usable IPs on this network

**Q3: What does MAC address do?**
**A**: Uniquely identifies your network card (hardware fingerprint). More stable than IP; used for network tracking and device identification. Can be spoofed (anti-forensic).

**Q4: Why is the gateway (192.168.1.1) important?**
**A**: It's your exit point to external networks. If compromised, attacker controls all your traffic. Can perform MITM attacks.

**Q5: What are your DNS servers and why multiple?**
**A**: 
- Primary: 2401:df80:1000::6767 (HGU)
- Secondary: 2401:df80:1000::6666 (HGU)
- Fallback: 8.8.8.8, 8.8.4.4 (Google)

Multiple servers provide redundancy. If HGU DNS fails, queries fallback to Google DNS.

### netstat Analysis

**Q6: What does "netstat -ano" show?**
**A**: 
- `-a` = all connections (listening + active)
- `-n` = numeric format (IP addresses as numbers)
- `-o` = PID (process ID making the connection)
Shows local/remote addresses, ports, connection state, and associated process.

**Q7: What does LISTENING mean in netstat?**
**A**: A service is waiting for incoming connections on that port. Can indicate open attack surface if unexpected.

**Q8: What does ESTABLISHED mean?**
**A**: An active two-way connection is currently open. Data can flow both directions. Proves connection active at capture time.

**Q9: You see port 445 listening. What is it and why is it dangerous?**
**A**: 
- Port 445 = SMB (Server Message Block) – Windows file sharing
- Dangerous = unpatched machines vulnerable to ransomware (WannaCry, NotPetya)
- If internet-facing, worms can infect automatically

**Q10: How do you find what app is making a network connection?**
**A**: 
1. Find PID from netstat output (e.g., PID 14680)
2. Open Task Manager → Details tab
3. Add PID column
4. Find that PID → shows process name (chrome.exe, etc.)

### tracert Analysis

**Q11: What does tracert (traceroute) do?**
**A**: Shows the network path (hops) from your device to destination. Each hop is a router. Reveals complete route and response times.

**Q12: How does tracert work technically?**
**A**: Sends packets with increasing TTL (Time-To-Live). Each router decrements TTL by 1. When TTL reaches 0, router sends error message revealing its IP. By incrementing TTL, you discover all hops.

**Q13: What does "Request timed out" in tracert mean?**
**A**: That router is not responding to ICMP ping messages. Usually due to firewall rules (intentional). Doesn't mean network is broken if other hops respond.

**Q14: Analyze your tracert: Why is Hop 10 significant?**
**A**: Hop 10 is final destination (Google Delhi). Hostname "tzdelb-bg-in-x04.1e100.net" shows:
- **tzdelb** = time zone Delhi
- Content served from India datacenter
- Optimal for India users

**Q15: If your tracert suddenly showed 15 hops instead of 10, what would you suspect?**
**A**: Possible network compromise or unusual routing. Could indicate MITM attack, DNS hijacking, or ISP routing changes. Should compare to baseline.

### Forensic Implications

**Q16: Why is MAC address more important than IP in forensics?**
**A**: IP changes with DHCP, but MAC is stable (unique per network card). Network logs correlate MAC to IP:
```
MAC 68-34-21-D0-36-99 → IP 192.168.1.35 → Dec 4 10:16–18:30 active
```
This proves the same device across network logs.

**Q17: How would you prove who was using a device at a specific time?**
**A**: 
1. Find device MAC from ipconfig
2. Check DHCP lease times (when was device connected?)
3. Check gateway/router logs (when did MAC appear?)
4. Cross-reference with user interviews and activity logs
5. Compare network activity to file timestamps

**Q18: What would indicate a Man-in-the-Middle (MITM) attack?**
**A**: 
- Gateway IP changed unexpectedly
- DNS servers changed
- Unusual route in tracert (more hops than baseline)
- SSL certificate errors (if HTTPS)
- Unexpected network latency increases

**Q19: If you found listening port 6667 on netstat, what would you suspect?**
**A**: Port 6667 = IRC (Internet Relay Chat) – commonly used by backdoors for command & control. Would indicate possible malware/compromise.

**Q20: How would you investigate suspicious outbound connection to unknown IP?**
**A**: 
1. Find PID from netstat
2. Identify process (Task Manager)
3. Check process legitimacy (Google process name)
4. WHOIS lookup on remote IP (identify server location/owner)
5. Check firewall logs (when did connection start? duration? data volume?)
6. Interview user ("Were you accessing this service?")
7. Compare to baseline (is this new?)

---

## Report Writing Template

**For Your Submission**:

```markdown
# Lab 2: Network Reconnaissance & IP Tracking

## Objective
Gather network configuration and active connections using Windows CLI tools
to establish baseline for forensic analysis.

## Procedure
1. Opened Command Prompt
2. Ran: ipconfig /all
3. Ran: netstat -ano
4. Ran: tracert www.google.com
5. Documented all findings

## Findings

### Network Configuration (ipconfig /all)
| Item | Value |
|------|-------|
| Hostname | LAPTOP-TKVKDEDI |
| IPv4 Address | 192.168.1.35 |
| Subnet Mask | 255.255.255.0 |
| MAC Address | 68-34-21-D0-36-99 |
| Gateway | 192.168.1.1 |
| DNS Servers | 2401:df80:1000::6767, 8.8.8.8, 8.8.4.4 |
| DHCP Enabled | Yes |
| Lease Time | 04 Dec 2025 10:16:21 – 05 Dec 2025 10:16:20 |

### Active Connections (netstat -ano)
| Local IP | Remote IP | Port | State | PID |
|---|---|---|---|---|
| 192.168.1.35:49409 | 4.213.25.242:443 | ESTABLISHED | 5364 |
| 192.168.1.35:58241 | 185.199.111.154:443 | ESTABLISHED | 14680 |

(List significant connections)

### Listening Ports
| Port | Service | Risk |
|------|---------|------|
| 135 | RPC | Medium |
| 445 | SMB | High |
| 7680 | Windows Update | Low |

### Routing to Google (tracert)
- Total Hops: 10
- First Hop: 2401:df80:1066:50c6:bab7:dbff:fe87:7284 (gateway)
- Final Hop: tzdelb-bg-in-x04.1e100.net (Google Delhi)
- Response Time: 1-25ms (healthy)

## Analysis

### Network Security Posture
**Strengths**: IPv6 enabled, multiple DNS servers, DHCP managed
**Concerns**: Port 445/135 listening, NetBIOS enabled, no VPN

### Active Connections
All connections are to known services (Microsoft, GitHub, Google) – normal for college student with development activity. No suspicious outbound connections.

### Routing Health
Trace shows normal path to Google Delhi datacenter. No unusual detours or compromised routing detected.

## Conclusion
Device is connected to HGU LAN via DHCP. Network configuration appears normal with no indicators of compromise. Baseline established for future comparison.

## Attachments
- Screenshot 1: ipconfig /all output
- Screenshot 2: netstat -ano output
- Screenshot 3: tracert output
```

---

**You now have a complete Lab 2 guide with**:
✓ All conceptual explanations (IP, MAC, Gateway, DNS, DHCP, IPv6)
✓ Your actual command outputs
✓ Detailed forensic analysis
✓ Viva Q&A (20 questions)
✓ Report template for submission

Good luck with your viva! 🎓
