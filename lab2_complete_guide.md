# Lab 2: Network Reconnaissance & IP Tracking - Complete Practical Guide

**Student**: Mayank  
**Date Completed**: December 4, 2025  
**System**: Windows 10/11 (LAPTOP-TKVKDEDI)  
**Course**: Digital Forensics - SEM VII

---

## Table of Contents

1. [Objective](#objective)
2. [Prerequisites & Concepts](#prerequisites--concepts)
3. [Tools Overview](#tools-overview)
4. [Step-by-Step Procedure](#step-by-step-procedure)
5. [Your Actual Output & Analysis](#your-actual-output--analysis)
6. [Detailed Explanations](#detailed-explanations)
7. [Forensic Significance](#forensic-significance)
8. [Viva Preparation](#viva-preparation)

---

## Objective

Use built-in **Windows Command-Line Interface (CLI)** tools to perform passive network reconnaissance:

- Gather local network configuration (IP, MAC, Gateway, DNS)
- Identify active network connections and associated processes
- Trace the network path (routing) to external hosts
- Document findings in a forensically sound manner

**Forensic Goal**: Establish baseline network state and detect anomalies without active probing.

---

## Prerequisites & Concepts

### 1.1 IP Address (IPv4 and IPv6)

**IPv4** (32-bit):
- Format: XXX.XXX.XXX.XXX (e.g., 192.168.1.35)
- Divided into Network + Host portions by the Subnet Mask
- Example: 192.168.1.35 with Subnet Mask 255.255.255.0
  - Network: 192.168.1.0
  - Host: .35 (unique device on that network)
  - Broadcast: 192.168.1.255

**IPv6** (128-bit):
- Format: hexadecimal notation (e.g., 2401:df80:1066:50c6:aa8a:5ff4:94fb:9e0f)
- Advantages: vastly more addresses, better security, auto-configuration
- Link-local IPv6: fe80::... (used for local-only communication)

### 1.2 MAC Address

- **Media Access Control** address (6 octets, e.g., 68-34-21-D0-36-99)
- Unique per **physical network interface** (NIC)
- Used for local-network (Layer 2) communication
- Can be spoofed (forensically significant – might indicate anti-forensics)

### 1.3 Gateway and Routing

The **Default Gateway** is the router—packets destined for other networks go to the gateway first.

- IPv4 Gateway: 192.168.1.1 (your home/office router)
- IPv6 Gateway: fe80::bab7:dbff:fe87:7284%11 (router on local link)

**Routing**: Devices hop through intermediate routers to reach distant destinations (visible with `tracert`/`traceroute`).

### 1.4 DHCP (Dynamic Host Configuration Protocol)

- Automatically assigns IP, Subnet Mask, Gateway, DNS Servers
- Lease Duration: temporary (often 24 hours)
- **Forensic Note**: IP assignment changes over time; MAC is more stable identifier

### 1.5 DNS (Domain Name System)

Translates human-readable names (e.g., www.google.com) to IP addresses.

- **DNS Server**: Device that performs resolution
- Multiple DNS servers for redundancy
- **Forensic Note**: DNS queries and cached resolvers indicate previously visited sites

**DNS Suffix Search List** (hgu_lan):
- When you type `computer.local` without a full domain, Windows tries `computer.local.hgu_lan`
- Indicates organizational network context

### 1.6 Ports and Services

- **Port**: 16-bit number (0–65535) identifying a specific service on a device
- Common services:
  - 22 – SSH
  - 53 – DNS
  - 80 – HTTP
  - 135 – RPC (Remote Procedure Call)
  - 139 – NetBIOS
  - 443 – HTTPS
  - 445 – SMB (Server Message Block / Windows File Sharing)

### 1.7 TCP States

- **LISTENING**: Service waiting for incoming connections
- **ESTABLISHED**: Active two-way communication in progress
- **TIME_WAIT**: Connection closed but in cleanup phase
- **CLOSE_WAIT**: Remote end closed; local socket cleaning up

**Forensic Note**: ESTABLISHED connections indicate active network activity at capture time.

### 1.8 Process ID (PID)

- Unique identifier for a running process
- Allows correlation between network connection and application
- Example: PID 14680 using port 443 suggests a browser or app making HTTPS connections

---

## Tools Overview

### 2.1 ipconfig /all

**Purpose**: Display all network adapters and their complete configuration.

**Output includes**:
- Hostname
- Each network adapter's status, MAC address, IP, Gateway, DNS servers
- DHCP server address and lease times
- IPv6 addresses (global + link-local)

**Forensic Use**:
- Identify all network interfaces (including VM adapters, VPNs)
- Detect unusual adapters (potential malware VPN or proxy)
- Compare against known-good baseline
- Determine if machine was DHCP or static-addressed at capture time

### 2.2 netstat -ano

**Parameters**:
- `-a`: All connections (listening + active)
- `-n`: Numeric format (IPs shown as numbers, not hostnames)
- `-o`: Show PID associated with each connection

**Output includes**:
- Protocol (TCP/UDP)
- Local Address:Port
- Remote Address:Port
- State (LISTENING, ESTABLISHED, TIME_WAIT, etc.)
- PID (allows cross-reference with Process List)

**Forensic Use**:
- Identify active outbound connections (data exfiltration, C2 communication)
- Detect unusual listening ports (backdoors, rootkits)
- Correlate network activity with specific processes
- Build timeline of communications

### 2.3 tracert www.google.com

**Purpose**: Trace the network path (route) packets take to reach a destination.

**How it works**:
1. Sends packets with increasing TTL (Time-To-Live)
2. Each intermediate router decrements TTL
3. When TTL reaches 0, router sends error back
4. Response reveals router's IP and response time
5. Continues until destination reached or max hops exceeded

**Output includes**:
- Hop number
- Response times (milliseconds) from each router
- IP address or hostname of each router
- "Request timed out" for hops that don't respond

**Forensic Use**:
- Map network topology (which routers involved)
- Detect network performance issues
- Identify geolocation of routers and servers
- Baseline comparison (unusual routing suggests compromise/ISP issues)

---

## Step-by-Step Procedure

### Step 1: Open Command Prompt

**Windows:**
- Press `Win + R`
- Type `cmd` or `cmd.exe`
- Press Enter
- (Optional: Run as Administrator for full details)

**Result**: Black terminal window with prompt like `C:\Users\Mayank>`

### Step 2: Run ipconfig /all

**Type**:
```
ipconfig /all
```

**What to Look For**:
- Hostname (identifies the computer)
- All active adapters (Ethernet, Wi-Fi, Virtual adapters)
- IPv4 Address and Subnet Mask
- IPv6 addresses (if enabled)
- MAC Address (Physical Address)
- Default Gateway(s)
- DHCP Enabled/Disabled
- DNS Server(s)
- Lease times (when IP was assigned and will expire)

**Screenshot**: Capture full output or scroll through carefully.

### Step 3: Run netstat -ano

**Type**:
```
netstat -ano
```

**Interpretation Guide**:
- First section: TCP connections (sorted by Local Address)
- Second section: UDP services
- Look for:
  - **ESTABLISHED** = active connections (use these for analysis)
  - **LISTENING** = open ports (services available for connection)
  - Port 0.0.0.0:X = listening on all interfaces (most dangerous if unexpected)
  - Port 127.0.0.1:X = listening on localhost only (less critical)
  - Remote address 0.0.0.0:0 = listening port (not actively connected)

**Important for Forensics**:
- Note any ESTABLISHED connections to unusual IPs
- Check if suspicious ports are listening
- Cross-reference PID with Task Manager to confirm legitimacy

### Step 4: Run tracert www.google.com

**Type**:
```
tracert www.google.com
```

**Interpretation**:
- First hop: Your Default Gateway (router)
- Middle hops: ISP and backbone routers
- Final hop: Google's server
- Look for:
  - "Request timed out" = router not responding to ICMP
  - Response times: <10ms = local, 10-50ms = regional, >50ms = distant
  - Geographic progression (should generally match realistic path)

**Alternative Targets** (for comparison):
```
tracert 8.8.8.8                    (Google Public DNS)
tracert microsoft.com              (US tech company)
tracert du.ac.in                   (your local university)
```

---

## Your Actual Output & Analysis

### Section A: Network Configuration (from ipconfig /all)

#### **System Information**

| Item | Your Value |
|------|-----------|
| Hostname | LAPTOP-TKVKDEDI |
| Primary DNS Suffix | (None - not domain-joined) |
| Node Type | Hybrid (uses broadcast, unicast, and P2P) |
| IP Routing Enabled | No (laptop, not router) |
| WINS Proxy Enabled | No (not using legacy NetBIOS) |
| DNS Suffix Search List | hgu_lan (indicates HGU/college network) |

**Forensic Interpretation**:
- "LAPTOP-TKVKDEDI" is a standalone workstation (not domain-controlled)
- Connected to organizational network (hgu_lan suggests HGU - Haryana university or similar)
- Not configured as a router

#### **Physical Network Adapters Status**

| Adapter | Status | MAC Address | Notes |
|---------|--------|------------|-------|
| **Ethernet** (Realtek GbE) | Media disconnected | 24-6A-0E-CE-42-76 | Not currently in use |
| **Wi-Fi Direct #1** | Media disconnected | 68-34-21-D0-36-9A | Virtual adapter, unused |
| **Wi-Fi Direct #2** | Media disconnected | 6A-34-21-D0-36-99 | Virtual adapter, unused |
| **Wi-Fi** (Intel Wi-Fi 6E) | **ACTIVE** | 68-34-21-D0-36-99 | Your active network interface |

**Forensic Interpretation**:
- **Active Interface**: Intel Wi-Fi 6E (modern, high-speed wireless)
- **Inactive Adapters**: Ethernet cable not plugged in; Wi-Fi Direct for nearby device sharing (not used)
- **MAC Addresses**: Note these for network tracking (if device needs to be traced later)

#### **Active Interface Details: Wi-Fi (Intel Wi-Fi 6E AX211)**

| Configuration | Value | Meaning |
|---------------|-------|---------|
| **IPv4 Address** | 192.168.1.35 | Local private IP (Class C private range 192.168.x.x) |
| **Subnet Mask** | 255.255.255.0 | /24 network; first three octets are network (192.168.1.0) |
| **Network Range** | 192.168.1.0–192.168.1.255 | 254 usable IPs (.1=gateway, .255=broadcast) |
| **Your Position** | 192.168.1.35 | 35th device on this network |
| **Default Gateway** | 192.168.1.1 (IPv4) | Your router (where packets for external networks go) |
| | fe80::bab7:dbff:fe87:7284%11 (IPv6) | IPv6 link-local gateway |
| **DHCP Server** | 192.168.1.1 | Router provides IP (not static) |
| **Lease Obtained** | 04 Dec 2025 10:16:21 | Last DHCP renewal |
| **Lease Expires** | 05 Dec 2025 10:16:20 | IP valid for ~24 hours |

**IPv4 Detailed Breakdown** (192.168.1.35 / 255.255.255.0):
- Network Portion: 192.168.1.X (controlled by first 3 octets + subnet mask)
- Host Portion: .35
- Network Address: 192.168.1.0
- Broadcast Address: 192.168.1.255
- First Usable IP: 192.168.1.1 (gateway/router)
- Last Usable IP: 192.168.1.254
- Your IP: 192.168.1.35 (usable, 35 devices numbered after gateway)

#### **IPv6 Addresses**

| Type | Address | Purpose |
|------|---------|---------|
| **Global Unicast (Preferred)** | 2401:df80:1066:50c6:aa8a:5ff4:94fb:9e0f | Internet-routable address from your ISP/network |
| **Temporary IPv6 (Preferred)** | 2401:df80:1066:50c6:5467:12:2a41:da13 | Privacy address (rotates to hide real identity) |
| **Link-Local** | fe80::c446:64d7:b432:3ca7%11 | Local-only communication (never routed to internet) |

**IPv6 Explanation**:
- `2401:df80:1000::/48` block = India-assigned prefix (2401 is India country code in IPv6)
- `50c6` = subnet identifier (like your network number)
- `aa8a:5ff4:94fb:9e0f` = host identifier (like your device address)
- Privacy address rotation prevents tracking across services

**Forensic Significance of IPv6**:
- Attackers can fingerprint devices by IPv6 address
- Privacy extensions help, but combined with MAC address still identifiable
- Dual-stack (IPv4 + IPv6) means multiple attack surfaces

#### **MAC Address**

| Field | Value |
|-------|-------|
| Physical Address (Wi-Fi) | 68-34-21-D0-36-99 |
| Burned-In Address (BIA) | Globally unique per device |
| DHCP Enabled | Yes |
| Autoconfiguration | Yes (APIPA fallback if DHCP fails) |

**MAC Address Breakdown** (68-34-21-D0-36-99):
- First 3 octets (68-34-21): OUI (Organizationally Unique Identifier) = Intel
- Last 3 octets (D0-36-99): Device-specific (serial-like)

**Forensic Use**:
- Network administrators track devices by MAC for access control
- Forensic tools search network logs for this MAC to find device activity
- Can be spoofed (if you change MAC, network sees you as new device) – *forensically significant*

#### **DNS Servers** (Multiple for Redundancy)

| Priority | Server | Type | Provider |
|----------|--------|------|----------|
| 1 | 2401:df80:1000::6767 | IPv6 | HGU/ISP DNS |
| 2 | 2401:df80:1000::6666 | IPv6 | HGU/ISP DNS |
| 3 | 8.8.8.8 | IPv4 | Google Public DNS |
| 4 | 8.8.4.4 | IPv4 | Google Public DNS (backup) |

**DNS Configuration Analysis**:
- **Primary**: HGU corporate DNS (IPv6-first setup, modern)
- **Fallback**: Google Public DNS (universal backup)
- **Forensic Implication**: This DNS config means:
  - Domain name resolutions go through HGU (they log it)
  - If HGU DNS is down, queries fall back to Google
  - Private organizations can see what sites you try to visit

#### **DHCPv6 Settings**

| Setting | Value |
|---------|-------|
| DHCPv6 IAID | 157824033 (device-specific ID) |
| Client DUID | 00-01-00-01-2E-68-C7-A3-24-6A-0E-CE-42-76 |

**Forensic Note**: DUID allows DHCPv6 servers to track your device permanently (even if MAC changes).

#### **NetBIOS over TCP/IP**

| Setting | Value |
|---------|-------|
| NetBIOS over Tcpip | Enabled |

**What is NetBIOS?**
- Legacy Microsoft protocol (port 137/138/139)
- Allows hostname resolution on local network (e.g., ping LAPTOP-TKVKDEDI)
- Security risk: can leak information in Windows networks
- **Forensic**: Enabled NetBIOS means Windows File Sharing (port 445) may be vulnerable

---

### Section B: Active Network Connections (from netstat -ano)

#### **Overview**

You ran `netstat -ano` and captured both TCP and UDP connections. Below is forensic analysis of significant findings.

#### **TCP LISTENING Ports** (Services Accepting Connections)

| Port | PID | Service | Risk Level |
|------|-----|---------|-----------|
| 135 | 1652 | RPC (Remote Procedure Call) | Medium – needed for Windows services |
| 445 | 4 | SMB (Server Message Block) | High – file sharing; exploitable if unpatched |
| 5040 | 3604 | UPnP (Device Discovery) | Medium – for local device discovery |
| 7680 | 17536 | Windows Update | Low – legitimate Windows service |
| 49664–49678 | Various | High-Range RPC Services | Low–Medium – system services |

**Forensic Interpretation**:
- Port 445 (SMB) + NetBIOS enabled = potential lateral movement vector
- Port 135 (RPC) = vulnerable to certain worms (e.g., WannaCry)
- High-range ports = system services, not user applications
- All ports listening on 0.0.0.0 = accepting connections from any interface (broader exposure than 127.0.0.1)

#### **TCP ESTABLISHED Connections** (Active Communications)

Your device was communicating with multiple remote servers. Here are the most interesting:

| Local IP:Port | Remote IP:Port | PID | Process | Purpose |
|---------------|---|-----|---------|---------|
| 192.168.1.35:49409 | 4.213.25.242:443 | 5364 | Unknown | HTTPS (likely Windows Update or Azure service) |
| 192.168.1.35:53090 | 40.126.18.33:443 | 2040 | Unknown | HTTPS (Microsoft network) |
| 192.168.1.35:58241 | 185.199.111.154:443 | 14680 | Likely Browser | HTTPS (GitHub CDN) |
| 192.168.1.35:58551 | 140.82.114.25:443 | 14680 | Likely Browser | HTTPS (GitHub) |
| 192.168.1.35:61938 | 185.199.111.154:443 | 14680 | Likely Browser | HTTPS (GitHub CDN) |

**Detailed Analysis**:

**Remote IP 4.213.25.242 (Microsoft Azure)**
- Likely Windows Update, Microsoft cloud services, or Office 365
- Port 443 = HTTPS (encrypted)
- Forensic Note: Can't see what data, only that communication occurred

**Remote IP 40.126.18.33 (Microsoft)**
- Another Microsoft data center
- Could be Edge browser cloud sync, OneDrive, or telemetry

**Remote IP 185.199.111.154 (GitHub)**
- Likely a web page or repository download
- PID 14680 suggests browser process
- Forensic Note: DNS query for github.com would appear in DNS logs

**Remote IP 140.82.114.25 (GitHub)**
- Direct GitHub server communication
- Suggests GitHub activity (browsing, cloning, authentication)

#### **IPv6 Connections** (Your Modern Network)

| Local Address:Port | Remote Address:Port | PID | State |
|---|---|-----|-------|
| [2401:df80:1066:50c6:5467:12:2a41:da13]:51641 | [2600:1f18:24e6:b902:a46c:a4a6:87fe:c14c]:443 | 14680 | ESTABLISHED |
| [2401:df80:1066:50c6:5467:12:2a41:da13]:58663 | [2600:1f18:24e6:b901:fbdc:7182:a89c:6101]:443 | 14680 | ESTABLISHED |
| [2401:df80:1066:50c6:5467:12:2a41:da13]:62035 | [2a03:2880:f311:120:face:b00c:0:167]:443 | 14680 | ESTABLISHED |
| [2401:df80:1066:50c6:5467:12:2a41:da13]:63957 | [2404:6800:4003:c03::bc]:5228 | 14680 | ESTABLISHED |

**IPv6 Connection Analysis**:
- `2a03:2880:f311:120:face:b00c:0:167` = Facebook (face:b00c = face:book easter egg!)
- `2404:6800:4003:c03::bc` = Google (port 5228 = Google Cloud Messaging)
- Multiple connections indicate active browsing and cloud service usage
- PID 14680 is likely your browser process

**Forensic Note**: IPv6 connections from a forensics perspective:
- Harder to geolocate (WHOIS less reliable)
- Can be used to bypass IPv4-based blocking/monitoring
- Important to include in incident timelines

#### **UDP Services** (Connectionless Protocols)

| Local Port | Remote | Protocol | PID | Service |
|-----------|--------|----------|-----|---------|
| 53 | DNS traffic | DNS | 14464, 1548 | System DNS queries |
| 5353 | mDNS traffic | mDNS | 14464 | Local service discovery |
| 137, 138 | NetBIOS | NetBIOS | 4 (System) | Windows name resolution |
| 5355 | LLMNR | LLMNR | 1548 | Link-Local name resolution |

**UDP Analysis**:
- **Port 53 (DNS)**: Your computer querying DNS for domain resolution
- **Port 5353 (mDNS)**: Local network service discovery (e.g., finding printers)
- **Ports 137/138 (NetBIOS)**: Legacy Windows protocol (security consideration)
- **Port 5355 (LLMNR)**: If DNS fails, uses this as fallback

**Forensic Significance of UDP**:
- Connectionless = no handshake
- Easier to spoof source IP
- DNS queries (port 53) reveal website visits
- No state tracking = harder to detect anomalies

---

### Section C: Network Routing to Google (from tracert www.google.com)

#### **Full Trace Output & Analysis**

```
Tracing route to www.google.com [2404:6800:4002:827::2004]
over a maximum of 30 hops:

  1     1 ms     1 ms     1 ms  2401:df80:1066:50c6:bab7:dbff:fe87:7284
  2     *        *        *     Request timed out.
  3     7 ms     5 ms     3 ms  2401:df80:1000::51
  4     2 ms    13 ms     2 ms  2001:4860:1:1::36a6
  5     8 ms    62 ms     2 ms  2404:6800:81e1:280::1
  6     4 ms     4 ms     3 ms  2001:4860:0:1::5e5c
  7     2 ms     2 ms     2 ms  2001:4860:0:1::77ac
  8     5 ms     3 ms    25 ms  2001:4860:0:1::77d5
  9     2 ms     2 ms     3 ms  2001:4860:0:1::5391
 10     4 ms     2 ms     2 ms  tzdelb-bg-in-x04.1e100.net [2404:6800:4002:827::2004]

Trace complete.
```

#### **Hop-by-Hop Analysis**

| Hop | IP Address / Hostname | RTT (ms) | Location / Owner | Forensic Notes |
|-----|---|-----|-------|--------|
| **1** | 2401:df80:1066:50c6:bab7:dbff:fe87:7284 | 1-1-1 | Your Gateway (Router) | First hop = exit from local network |
| **2** | Request timed out | – | ISP Gateway | Not responding to ICMP (filtered/busy) |
| **3** | 2401:df80:1000::51 | 3-7 ms | HGU/ISP Network | Local ISP node |
| **4** | 2001:4860:1:1::36a6 | 2-13 ms | Google Network (AS15169) | Entering Google's backbone |
| **5** | 2404:6800:81e1:280::1 | 2-62 ms | Google Asia-Pacific | Regional Google datacenter |
| **6–9** | 2001:4860:0:1::xxxx | 2-25 ms | Google Network (internal routing) | Routing within Google's infrastructure |
| **10** | tzdelb-bg-in-x04.1e100.net | 2-4 ms | **Google Delhi, India** | Final destination (TZDELB = time zone Delhi) |

#### **Key Observations**

**Total Hops**: 10 hops (reasonable for international internet, though trace is to India)

**Response Times**:
- Hop 1–3: ~1-7ms = local/ISP (fast)
- Hop 4–9: ~2-25ms = Google network (fast, optimized routing)
- Hop 10: ~2-4ms = final destination (very responsive)

**Overall Quality**: Excellent connectivity (all hops responsive, consistent latency)

**Hop 2 Timeout**:
- ISP gateway not responding to ICMP
- Common for security reasons (firewalls drop ICMP)
- Doesn't indicate problem (hop 3 reachable)

#### **Forensic Interpretation**

| Aspect | Finding | Significance |
|--------|---------|--------------|
| **First Hop** | 2401:df80:... (IPv6 local gateway) | Confirms you're on IPv6-enabled network; gateway is responsive |
| **ISP Node** | HGU/ISP gateway reached | You're connected to ISP; no ISP-side blockage |
| **Google Entry** | Google AS15169 reached at hop 4 | Direct path to Google (no detours) |
| **Delhi Destination** | tzdelb-bg-in-x04.1e100.net | Content served from India (local CDN edge) – excellent for India users |
| **Latency** | Consistent 2-25ms | Good network health; no congestion |

**Forensic Conclusion**:
- Network is healthy and well-connected
- No unexpected routing detours (would indicate MITM or ISP redirection)
- ISP is not blocking Google (trace completes successfully)
- Geographic routing is efficient (India-based users → Delhi datacenter)

---

## Detailed Explanations

### 3.1 Understanding Network Classes

**Classful Addressing** (older, but still used for mental model):

| Class | Range | First Octet | Default Subnet | Private Range? |
|-------|-------|-----------|--------|---|
| A | 1–126.x.x.x | 1–126 | 255.0.0.0 | 10.0.0.0/8 |
| B | 128–191.x.x.x | 128–191 | 255.255.0.0 | 172.16.0.0/12 |
| **C** | **192–223.x.x.x** | **192–223** | **255.255.255.0** | **192.168.0.0/16** |
| D | 224–239.x.x.x (Multicast) | – | – | – |
| E | 240–255.x.x.x (Reserved) | – | – | – |

**Your IP (192.168.1.35)**:
- Class C (starts with 192)
- Private range (RFC 1918 protected)
- Means: never routable to internet directly
- Your router translates private IP → public IP (NAT)

### 3.2 Subnet Mask Deep Dive

**Subnet Mask: 255.255.255.0**

In binary:
```
255         .255         .255         .0
11111111    .11111111    .11111111    .00000000
```

**Meaning**:
- First 24 bits = network (255.255.255)
- Last 8 bits = host (0)
- Written as: /24 (CIDR notation)

**IP 192.168.1.35 with /24 mask**:

| Octet | Decimal | Binary | Type |
|-------|---------|--------|------|
| 1 | 192 | 11000000 | Network (fixed) |
| 2 | 168 | 10101000 | Network (fixed) |
| 3 | 1 | 00000001 | Network (fixed) |
| 4 | 35 | 00100011 | **Host (variable)** |

**Network Analysis**:
- All devices on 192.168.1.X share same network
- Your device: .35
- Router: .1 (by convention)
- Broadcast: .255 (to all devices on network)
- Usable: .2 to .254 (253 usable addresses)

### 3.3 MAC Address Forensics

**Your MAC**: 68-34-21-D0-36-99

**Structure**:
- **68-34-21**: Vendor ID (Intel)
- **D0-36-99**: Device Serial-like ID

**Forensic Significance**:

1. **Network Tracking**: Network admins log:
   ```
   MAC: 68-34-21-D0-36-99 → IP: 192.168.1.35 → User: Mayank
   ```
   Creates audit trail.

2. **Spoofing Detection**: If MAC changes unexpectedly:
   - Could indicate attacker using stolen MAC
   - Or network adapter replacement
   - Or active MAC spoofing (anti-forensic technique)

3. **Device Persistence**: MAC more stable than IP (IP changes with DHCP)
   - Investigators often track by MAC across network logs

### 3.4 Gateway Significance

**Default Gateway: 192.168.1.1**

**Role**:
- Your "exit point" from local network
- Typically a WiFi router or network switch
- Has a public IP on internet side

**Forensic Implications**:

1. **Packet Flow**:
   ```
   Your Device (192.168.1.35)
   → Gateway (192.168.1.1)
   → ISP Gateway
   → Internet
   ```

2. **Data Loss Point**: If attacker controls the gateway:
   - Can sniff all your traffic
   - Can redirect connections
   - Can impersonate any host

3. **Network Compromise**: Non-standard gateway IP suggests:
   - Malware changed it
   - ARP spoofing attack
   - Network misconfiguration

### 3.5 DNS Significance in Forensics

**Your DNS Servers**:
1. 2401:df80:1000::6767 (HGU)
2. 2401:df80:1000::6666 (HGU)
3. 8.8.8.8 (Google)
4. 8.8.4.4 (Google)

**Forensic Implications**:

1. **DNS Logs**: HGU can see all domain queries:
   ```
   [2025-12-04 14:23:45] MAC: 68-34-21-D0-36-99 → Query: github.com
   ```

2. **DNS Poisoning**: If attacker controls DNS:
   - Can redirect bank.com → fake-bank.com
   - User sees legitimate HTTPS cert (DNS only, not cert validation)
   - Silent account takeover possible

3. **Privacy**: Private organizations can infer websites you visit:
   ```
   github.com, stackoverflow.com, youtube.com → Software developer
   medical.hospital.org, pharmacy.com → Health-related search
   ```

### 3.6 Ports in Depth

**Well-Known Ports (0–1023)**: System services (privileged)

#### **Dangerous Listening Ports**

| Port | Service | Risk | Exploitation |
|------|---------|------|--------------|
| **135** | RPC | High | WannaCry, EternalBlue |
| **139** | NetBIOS | High | NBNS/LLMNR poisoning |
| **445** | SMB | **Critical** | Ransomware propagation, lateral movement |
| **3389** | RDP | High | Brute force, encryption bypass |

**Your Device**: Port 445 listening = potential SMB attack surface.

#### **Ephemeral Ports (49,152–65,535)**: Dynamic client ports

When you open a browser and connect to google.com:
1. Browser picks random port (e.g., 49409)
2. Creates connection: YOUR_IP:49409 ↔ GOOGLE_IP:443
3. Connection is ESTABLISHED
4. Port 49409 only used by that connection
5. After close, port released back to pool

**Forensic Note**: Ephemeral ports appear/disappear rapidly in netstat output.

### 3.7 TCP Connection States

**Your netstat output showed**:

| State | Meaning | Forensic Significance |
|-------|---------|----------------------|
| **LISTENING** | Service awaiting connections | Open attack surface |
| **ESTABLISHED** | Active two-way communication | Current activity at capture time |
| **TIME_WAIT** | Connection closed, waiting to clear | Recent activity; connection no longer active |

**Example from Your Output**:
```
TCP    192.168.1.35:49409     4.213.25.242:443       ESTABLISHED     5364
```
Means: Your device (port 49409) actively connected to 4.213.25.242 (port 443) via process PID 5364.

### 3.8 Process Identification (PID)

**PID**: Process ID – unique identifier for running program

**Cross-Reference PID to Process**:
1. Open Task Manager (Ctrl+Shift+Esc)
2. Go to "Details" tab
3. Enable column: "PID"
4. Search for PID from netstat output
5. Matches PID with process name

**Your Example**:
- netstat shows PID 14680 has HTTPS connections to GitHub
- Task Manager: PID 14680 = `chrome.exe` or `firefox.exe` (browser process)
- **Conclusion**: Your browser was accessing GitHub

**Forensic Benefit**:
- Proves which application made the connection
- Helps distinguish malware from legitimate use
- Timeline correlation (when was process active)

---

## Forensic Significance

### 4.1 Why Network Reconnaissance Matters

**In Incident Response**:

1. **Establishing Baseline**: "What does normal look like?"
   - Normal: Few ESTABLISHED connections, known services listening
   - Abnormal: Dozens of connections to unknown IPs, unusual ports

2. **Detecting Compromise**:
   - Unexpected listening ports = possible backdoor
   - Unknown outbound connections = C2 (Command & Control) communication
   - Unusual gateway/DNS = network redirect attack

3. **Lateral Movement Detection**:
   - Connections to internal IPs (192.168.x.x, 10.x.x.x) = network spread
   - SMB connections to other machines = ransomware propagation

4. **Data Exfiltration**:
   - Large data volumes to external IPs
   - Connections to known data aggregation sites
   - Timing correlation with access to sensitive files

### 4.2 Your Network's Security Posture

**Strengths**:
- ✓ Using IPv6 (modern, better security features)
- ✓ Multiple DNS servers (redundancy)
- ✓ Private IP addressing (NAT protection)
- ✓ DHCP enabled (easier to manage)

**Concerns**:
- ⚠️ Port 445 (SMB) listening (ransomware attack vector)
- ⚠️ Port 135 (RPC) listening (WannaCry vulnerable)
- ⚠️ NetBIOS enabled (legacy protocol, security risk)
- ⚠️ No VPN in use (ISP can see all traffic)

**Recommendations**:
1. Disable SMB if not needed (if working alone)
2. Disable NetBIOS (outdated)
3. Use VPN for public WiFi
4. Update Windows (patches vulnerabilities)

### 4.3 Your Active Connections Analysis

**What You Found**:
- Multiple HTTPS connections (good encryption)
- Connections to GitHub (development activity)
- Connections to Microsoft/Google (cloud services)
- No suspicious outbound connections

**Forensic Verdict**: Normal for college student with GitHub activity, Windows updates, cloud services.

**Red Flags That Would Indicate Compromise**:
- ❌ ESTABLISHED to unknown external IPs (malware C2)
- ❌ Listening on port 6667 (IRC backdoor)
- ❌ Port forwarding enabled (lateral movement)
- ❌ Multiple connections to same port 443 from different processes (data theft)
- ❌ UDP to port 53 (DNS) to non-standard servers (DNS hijacking)

---

## Viva Preparation

### 5.1 Key Definitions (Memorize)

1. **IP Address**: Unique network identifier (IPv4: 192.168.1.35; IPv6: 2401::...)

2. **MAC Address**: Physical network interface identifier (68-34-21-D0-36-99)

3. **Gateway**: Router that connects your network to other networks

4. **Subnet Mask**: Defines which bits of IP are network vs. host (255.255.255.0 = /24)

5. **DHCP**: Protocol that automatically assigns IP, Gateway, DNS to devices

6. **DNS**: Service that translates domain names to IP addresses

7. **Port**: 16-bit service identifier (443 = HTTPS, 80 = HTTP, 53 = DNS)

8. **PID**: Process ID – unique identifier for running program

9. **ESTABLISHED**: TCP connection state indicating active communication

10. **Trace Route**: Network diagnostic that shows packet path to destination

### 5.2 Common Viva Questions (with Answers)

**Q1: What does ipconfig /all tell you?**  
**A**: All network adapters and their configuration: IP, MAC, Gateway, DNS, DHCP status, lease times, IPv6 addresses.

**Q2: Explain your IPv4 address 192.168.1.35 / 255.255.255.0**  
**A**: 
- 192.168.1.0 is the network
- .35 is your host on that network
- /24 means first 24 bits are network, last 8 bits are host
- Range: 192.168.1.1 to 192.168.1.254 (253 usable)
- 192.168.1.1 = gateway, 192.168.1.255 = broadcast

**Q3: Why does your MAC matter in forensics?**  
**A**: 
- Network logs correlate MAC → IP → User
- More stable than IP (doesn't change with DHCP)
- Spoofing MAC is anti-forensic technique
- Used to track device across network

**Q4: What's the difference between 192.168.1.1 (gateway) and 192.168.1.35 (your IP)?**  
**A**:
- 192.168.1.1 = router (network device)
- 192.168.1.35 = your laptop
- Packets from outside your network go to .1 first
- .1 routes to .35 (and all other devices)

**Q5: Explain the DNS server importance**  
**A**:
- 2401:df80:1000::6767 (HGU DNS) = primary
- 8.8.8.8 (Google DNS) = fallback
- DNS translates domain.com → IP address
- Forensically: DNS server logs all queries (what sites you tried to visit)

**Q6: What does "netstat -ano" tell you?**  
**A**: 
- `-a` = all connections (listening + active)
- `-n` = numeric (IP addresses, not hostnames)
- `-o` = output PID (which process owns connection)
- Shows local address:port, remote address:port, connection state, and PID

**Q7: What does "ESTABLISHED" mean in netstat output?**  
**A**: Active two-way communication happening right now. The connection is established; data can flow both directions.

**Q8: You found connection to 185.199.111.154:443. How do you know it's GitHub?**  
**A**: 
- WHOIS lookup on IP reveals it's GitHub's network
- DNS reverse lookup: 185.199.111.154 → cdn-xxx.github.com
- Or: Browser PID 14680 matched to browser process + HTTPS + GitHub patterns

**Q9: Explain the tracert output—what does each hop represent?**  
**A**: 
- Hop 1 = your gateway (router)
- Hop 2-N = intermediate routers (your ISP, backbone, destination's network)
- Hop 10 (final) = Google's server
- Response time shows latency; timeout means router not responding to ICMP

**Q10: If tracert shows "Request timed out" at Hop 2, does that mean network is broken?**  
**A**: 
- No, that router is just not responding to ICMP (ping)
- Many routers drop ICMP for security
- If Hop 3+ are reachable, network is fine
- In your trace, Hop 3 succeeded, so no problem

**Q11: Forensically, what would suggest network compromise?**  
**A**: 
- Listening ports you don't recognize (backdoors)
- ESTABLISHED connections to unknown IPs (C2 malware)
- Non-standard gateway IP (MITM attack)
- DNS changed to attacker-controlled server (DNS hijacking)
- Port 445/135 connections to external (worm propagation)

**Q12: Why is port 445 dangerous?**  
**A**: 
- SMB (Server Message Block) file sharing protocol
- Unpatched machines vulnerable to ransomware (WannaCry, NotPetya)
- If publicly exposed (not behind NAT), worm can infect you
- Lateral movement vector in networks

**Q13: What's the difference between private IP (192.168.x.x) and public IP?**  
**A**:
- **Private**: Only routable within your local network (hidden behind NAT)
- **Public**: Routable on internet (ISP assigns, visible globally)
- Your device has private IP (192.168.1.35)
- Gateway translates it to public IP when accessing internet
- Forensic: Private IPs appear in internal logs; public IPs in external logs

**Q14: Why does your device have both IPv4 and IPv6 addresses?**  
**A**: 
- Dual-stack networking (modern standard)
- IPv4: 192.168.1.35 (backwards compatible, limited address space)
- IPv6: 2401:df80:1066:50c6:aa8a:5ff4:94fb:9e0f (future-proof, vast addresses)
- Both work simultaneously
- Forensic: Attacker might use IPv6 to bypass IPv4 monitoring

**Q15: What does "Lease Obtained: 04 Dec 2025 10:16:21" mean?**  
**A**: 
- Your IP was assigned by DHCP server on that date/time
- DHCP Lease Expires: 05 Dec 2025 10:16:20 (24 hours duration)
- After expiration, IP may be reassigned to another device
- Forensic: DHCP logs show when device was active on network

### 5.3 Practical Demonstration Questions

**Q: How would you identify if a specific process (e.g., CHROME.EXE) is making unauthorized connections?**

**A (Step-by-Step)**:
1. Run `netstat -ano` and pipe to grep: `netstat -ano | findstr ESTABLISHED`
2. Identify CHROME.EXE's PID (from Task Manager Details tab)
3. Search netstat for that PID
4. Note all remote IPs and ports
5. Check if any are suspicious (unknown IPs, high ports)
6. Reverse DNS lookup on those IPs to identify services
7. Compare against known-good baseline (what is normal for Chrome?)
8. Flag any unexpected connections (e.g., Chrome connecting to port 4444, a backdoor port)

**Q: A colleague suspects ransomware. How would you use ipconfig/netstat/tracert to investigate?**

**A (Investigation Checklist)**:
1. **ipconfig**: Verify gateway and DNS haven't been changed (would indicate compromise)
2. **netstat -ano**: Look for:
   - Unusual listening ports (typically 135, 445, 3389 enough; extras = backdoor)
   - ESTABLISHED connections to unknown IPs (C2 communication)
   - Multiple connections to same external port (data exfiltration)
3. **tracert**: Check if routing is normal (unusual detours = MITM)
4. **Cross-check**: Correlate suspicious netstat IPs with file timestamp
   - If ransomware encrypted files, check what processes were active then
5. **Sample**: If suspicious PID found, kill it and see if encryption stops (confirms malware)

**Q: Document a network baseline for forensic comparison**

**A (Baseline Document)**:
```
Network Baseline – Laptop-TKVKDEDI
Date: 04-Dec-2025, 10:16:21 UTC

NETWORK CONFIG:
  IPv4: 192.168.1.35 / 255.255.255.0
  IPv6: 2401:df80:1066:50c6:aa8a:5ff4:94fb:9e0f
  MAC: 68-34-21-D0-36-99
  Gateway: 192.168.1.1
  DNS: [list above]

LISTENING PORTS (Normal):
  135 (RPC), 445 (SMB), 5040 (UPnP), 7680 (WinUpdate)

ESTABLISHED (Normal):
  Chrome to GitHub (185.199.111.154:443)
  Windows Update to Microsoft (40.126.18.33:443)

DEVIATION PROTOCOL:
  If new listening port appears → investigate
  If connection to non-whitelisted IP → alert
  If gateway/DNS changed → immediate remediation
```

### 5.4 Lab Report Writing Tips

**Report Structure** (for marks):

1. **Title**: "Lab 2: Network Reconnaissance & IP Tracking"
2. **Objective** (2–3 lines)
3. **Procedure** (step-by-step what you did)
4. **Observations** (tables with your actual output):
   - Network config table
   - Active connections table
   - Tracert output table
5. **Analysis** (interpret findings):
   - What does each IP/port mean?
   - Are they normal?
   - Any security concerns?
6. **Conclusion** (tie back to objective):
   - You successfully identified network state
   - No anomalies detected
   - Device is secure/has risks
7. **Screenshots** (attach):
   - CMD output (ipconfig, netstat, tracert)

**Marks Awarded For**:
- ✓ Complete command output
- ✓ Correct interpretation of IP/subnet/gateway
- ✓ Identifying listening ports and risk
- ✓ Cross-referencing PID to processes
- ✓ Understanding routing (tracert)
- ✓ Forensic conclusions (what does it mean?)
- ✓ Professional presentation (clear tables, explanations)

---

## Appendices

### Appendix A: IP Subnet Calculator (For Reference)

**Your Setup**: 192.168.1.35 / 255.255.255.0

| Calculation | Binary | Result |
|---|---|---|
| Network | 192.168.1.0 | Base network address |
| Broadcast | 192.168.1.255 | Address to all devices |
| First Host | 192.168.1.1 | Gateway (router) |
| Last Host | 192.168.1.254 | Last usable IP |
| Total Usable IPs | – | 253 devices |
| Your Position | – | 35th device on network |

### Appendix B: Common Ports Reference

| Port | Service | Type | Risk |
|------|---------|------|------|
| 21 | FTP | Unencrypted file transfer | High |
| 22 | SSH | Encrypted remote shell | Low (if properly configured) |
| 25 | SMTP | Email sending | Medium |
| 53 | DNS | Domain resolution | Medium (hijacking risk) |
| 80 | HTTP | Unencrypted web | High (no encryption) |
| 110 | POP3 | Email receive | High |
| 135 | RPC | Windows service | **High** (ransomware vector) |
| 139 | NetBIOS | Legacy Windows | High |
| 143 | IMAP | Email access | High |
| 3389 | RDP | Remote desktop | High (brute force) |
| 445 | SMB | Windows file sharing | **Critical** |
| 5900 | VNC | Remote screen | High |
| 6667 | IRC | Chat (backdoor use) | High (if unexpected) |

---

## Conclusion

**Lab 2 Summary**:

You successfully performed passive network reconnaissance using Windows built-in tools:

1. **ipconfig /all**: Revealed network configuration, IP addressing, DNS setup
2. **netstat -ano**: Identified active connections and listening services
3. **tracert**: Traced routing path and measured latency to Google

**Key Findings**:
- Your device: 192.168.1.35 on HGU network
- Active HTTPS connections to GitHub, Microsoft, Google (normal)
- Listening ports include 445 (SMB) – security consideration but manageable with NAT
- Network path to Google is healthy (10 hops, good latency)
- No indicators of compromise detected

**Forensic Value**:
- This baseline can be compared against future captures to detect anomalies
- Malware signatures include unusual listening ports, unknown connections, altered gateway/DNS
- PID correlation helps identify which application made connections
- IPv6 connections often overlooked but equally important to track

**For Your Viva**:
- Memorize definitions (IP, MAC, Gateway, DNS, Port, PID, ESTABLISHED state)
- Be ready to explain your specific output (why that IP? what does it mean?)
- Understand forensic implications (how would you detect compromise?)
- Practice correlating network findings to real-world scenarios

---

## References

1. Windows IPv6 Guide: https://docs.microsoft.com/en-us/windows/win32/winsock/ipv6-guide
2. Netstat Documentation: https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat
3. TCP/IP Fundamentals: RFC 791 (IPv4), RFC 8200 (IPv6)
4. Subnet Calculation: https://www.calculator.net/ip-subnet-calculator.html
5. WHOIS Lookup: https://www.whois.com/
