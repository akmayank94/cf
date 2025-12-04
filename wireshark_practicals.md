# Wireshark Practicals – Complete Guide for Cyber Forensics Lab

Author: Generated for student learning and viva preparation  
Scope: 4 Wireshark Practicals (Protocol Hierarchy, Endpoints & Conversations, I/O Graphs, HTTP/DNS Activity)

---

## 0. Introduction: Why Wireshark Matters in Cyber Forensics

Wireshark is a **network packet sniffer and analyzer**. It captures raw packets moving over a network and lets an investigator:

- See **which protocols** are in use
- Identify **which devices** (endpoints) are talking to each other
- Measure **how much traffic** each protocol or device generates
- Visualize **traffic spikes** and anomalies over time
- Reconstruct **user activity** (web browsing, DNS lookups, file downloads)

In cyber forensics, Wireshark helps answer questions like:

- Which websites were visited?
- Which IPs did a machine communicate with?
- Was a large file exfiltrated at a specific time?
- What protocols and ports were involved in suspicious activity?

This guide covers 4 core practicals that build these skills step‑by‑step.

---

## 1. Core Networking Concepts (Prerequisites for All Practicals)

Before using Wireshark, understand these key ideas.

### 1.1 Packets

A **packet** is a small unit of data sent over a network. It is like an envelope in postal mail:

- **Header** – meta‑information (source IP, destination IP, protocol, ports)
- **Payload** – actual data (web page content, image, video chunk, etc.)

Every action (browsing, streaming, downloading) is made up of many packets.

### 1.2 Network Protocols and Layers (TCP/IP View)

Protocols are **rules of communication**. They are organized in layers:

1. **Network Layer (IP)** – routing using IP addresses (e.g., 192.168.1.10)
2. **Transport Layer (TCP/UDP)**
   - **TCP** – reliable, ordered, connection‑oriented (web, email)
   - **UDP** – fast, connectionless (DNS, streaming, VoIP)
3. **Application Layer** – human‑visible services (HTTP, HTTPS, DNS, FTP, SMTP, etc.)

Packets usually contain **multiple stacked protocols** – e.g., HTTP over TCP over IP.

### 1.3 Packets vs Bytes

- **Packets** = number of discrete transmissions
- **Bytes** = total volume of data

Example:

- DNS: many **small** packets → high packet count, low bytes
- Video stream: fewer **large** packets → moderate packet count, very high bytes

This explains why top protocols by packets and by bytes can differ.

### 1.4 IP Addresses, Endpoints, and Conversations

- **Endpoint** – a device on the network, identified by IP (e.g., 192.168.1.5)
- **Conversation** – communication between two endpoints (IP1 ↔ IP2, with ports)

Wireshark can list all endpoints and all conversations between them.

### 1.5 Ports and Common Services

Ports are like apartment numbers on a building (a machine). Common ones:

- 21 – FTP
- 22 – SSH
- 25 – SMTP
- 53 – DNS
- 80 – HTTP (unencrypted web)
- 110 – POP3
- 143 – IMAP
- 443 – HTTPS (encrypted web)
- 445 – SMB (Windows file sharing)

Knowing ports helps identify the **application/service** used in a conversation.

### 1.6 HTTP vs HTTPS (Very Important)

- **HTTP (port 80)** – unencrypted, full content visible in Wireshark
- **HTTPS (port 443)** – encrypted (TLS/SSL); only IPs, ports, and a little metadata are visible

For forensics, HTTP gives **full visibility**, HTTPS mainly gives **metadata and timing**.

### 1.7 DNS and A Records

DNS translates domain names to IPs.

- **Query Name** – domain asked (e.g., `neverssl.com`)
- **Query Type** – usually **A** (IPv4 address)
- **Response** – IP address or error (e.g., NXDOMAIN = domain does not exist)

DNS queries often happen **before** HTTP requests and show user intent.

### 1.8 TCP 3‑Way Handshake

Before sending HTTP data, TCP establishes a connection:

1. **SYN** – client → server: "I want to connect"
2. **SYN‑ACK** – server → client: "OK, I accept"
3. **ACK** – client → server: "Acknowledged, start data"

In Wireshark, these packets appear immediately before an HTTP GET.

---

## 2. Practical 1 – Protocol Hierarchy Analysis

**Objective:** Analyze a capture file to determine the most prevalent protocols and compare how they rank by **packets** vs **bytes**.

### 2.1 What You Learn

- How to capture live traffic
- How to use **Statistics → Protocol Hierarchy**
- Why some protocols dominate packet count but not byte count

### 2.2 Step‑by‑Step Procedure

1. **Start Wireshark and Begin Capture**
   - Open Wireshark.
   - Select your active interface (Wi‑Fi/Ethernet) with live graph.
   - Click the **blue shark fin** to start capturing.
   - Use your computer **normally for 5 minutes** (browse, watch a short video, etc.).

2. **Stop Capture**
   - Click the **red square/stop** button.

3. **Open Protocol Hierarchy**
   - Go to **Statistics → Protocol Hierarchy**.
   - A window shows a tree of protocols with columns: Packets, % Packets, Bytes, % Bytes.

4. **Identify Top Protocols by Packets**
   - Sort or visually inspect the **% Packets** column.
   - Note the **top 3 protocols** by packet percentage.
   - Example table for your report:

     | Rank | Protocol | % Packets |
     |------|----------|-----------|
     | 1 | [Protocol‑A] | [value] |
     | 2 | [Protocol‑B] | [value] |
     | 3 | [Protocol‑C] | [value] |

5. **Identify Top Protocols by Bytes**
   - Look at **% Bytes** column.
   - Note the **top 3 protocols** by byte percentage.

     | Rank | Protocol | % Bytes |
     |------|----------|---------|
     | 1 | [Protocol‑X] | [value] |
     | 2 | [Protocol‑Y] | [value] |
     | 3 | [Protocol‑Z] | [value] |

6. **Analyze Differences**
   - Compare the two lists.
   - Common explanation pattern:
     - Small‑payload protocols (like DNS) may have **many packets** but **few bytes**.
     - Data‑heavy protocols (like HTTPS or video streaming) may have **fewer packets** but **large payloads**, dominating bytes.

### 2.3 Points to Write in Observation/Analysis

- Mention which protocols appeared at which layers (IP, TCP/UDP, HTTP, DNS, TLS).
- Explain nesting: HTTP inside TCP inside IP – therefore their percentages can overlap.
- Discuss why IP often has ~99% packets and bytes (almost all traffic uses IP).
- Give at least one concrete example comparing DNS vs HTTPS or similar.

### 2.4 Common Viva Questions (with Short Answers)

- **Q:** Why does IP usually have the highest % packets?
  - **A:** Almost every captured packet uses IP as the network layer protocol.

- **Q:** Why might DNS be high in % packets but low in % bytes?
  - **A:** DNS queries are small; each packet carries very little data.

- **Q:** What is the difference between packet count and byte count?
  - **A:** Packet count = number of individual transmissions; byte count = total data volume.

- **Q:** Why are some protocols shown as sub‑entries in the hierarchy?
  - **A:** Because protocols are layered; application protocols (HTTP, DNS) sit inside transport/IP layers.

---

## 3. Practical 2 – Endpoints & Conversations

**Objective:** Identify all unique devices (endpoints) on the network and analyze communication **conversations** between them.

### 3.1 What You Learn

- How to list **endpoints** (all IPs seen in capture)
- How to list **conversations** (pairs of endpoints talking to each other)
- How to identify the busiest IPs and conversations
- How to infer application type from port numbers

### 3.2 Step‑by‑Step Procedure

1. **Capture Traffic (5 minutes)**
   - Start a new capture on the active interface.
   - Generate traffic (browse, stream, download) for at least 5 minutes.
   - Stop the capture.

2. **View Endpoints**
   - Go to **Statistics → Endpoints**.
   - Click the **IPv4** tab.
   - The table shows each IP with columns such as Packets, Bytes, Tx/Rx.

3. **Record Top 5 IP Endpoints by Packets**
   - Sort by **Packets** (descending) if needed.
   - Copy top 5 entries into your report:

     | Rank | IP Address | Packets | Bytes | Notes |
     |------|-----------|---------|-------|-------|
     | 1 | [IP‑1] | [#] | [#] | Likely your machine/router |
     | 2 | [IP‑2] | [#] | [#] | External server, etc. |
     | 3 | ... | ... | ... | ... |
     | 4 | ... | ... | ... | ... |
     | 5 | ... | ... | ... | ... |

   - Add notes like "local host", "router", "Google server", etc., if known.

4. **View Conversations**
   - Go to **Statistics → Conversations**.
   - Click the **TCP** tab (focus on TCP conversations).
   - Each row represents a **pair of IPs and ports** communicating.

5. **Find Busiest Conversation (by Packets)**
   - Sort by **Packets** column.
   - Pick the top entry.
   - Record for report:

     - **IP Address 1**: [value]
     - **Port 1**: [value]
     - **IP Address 2**: [value]
     - **Port 2**: [value]
     - **Total Packets**: [value]

6. **Infer the Application/Service from Port Numbers**
   - Check which well‑known ports appear (21, 22, 53, 80, 443, etc.).
   - Example in report:

     > Conversation between 192.168.1.5:54321 and 142.250.190.78:443 with highest packet count suggests **HTTPS web browsing**, since port 443 is typically used by HTTPS.

### 3.3 Typical Endpoint/Conversation Interpretation

- Your own machine often has most packets.
- Your router or gateway IP appears frequently.
- External IPs may belong to Google, CDNs, DNS resolvers, etc.
- Conversations with port 80/443 are usually web traffic.

### 3.4 Viva Questions (with Brief Answers)

- **Q:** What is an endpoint in Wireshark?
  - **A:** A unique network address (e.g., IP address) representing one device in the capture.

- **Q:** What is a conversation?
  - **A:** A flow of packets exchanged between two specific endpoints/ports.

- **Q:** How can you guess the application/service from a conversation?
  - **A:** By looking at well‑known port numbers (80=HTTP, 443=HTTPS, 21=FTP, 22=SSH, 53=DNS, etc.).

- **Q:** Why focus on TCP in Conversations tab?
  - **A:** TCP is connection‑oriented; conversations are easier to interpret and track.

---

## 4. Practical 3 – I/O Graph and Traffic Spike Analysis

**Objective:** Use the **I/O Graph** to visualize network traffic over time and identify a **significant traffic spike**, then analyze which protocol caused it.

### 4.1 What You Learn

- How to graph packet/byte rates over time
- How to visually identify spikes or anomalies
- How to correlate spikes with user actions
- How to filter packets around a spike and re‑analyze them

### 4.2 Step‑by‑Step Procedure

1. **Plan a High‑Traffic Action**
   - Choose an action that will clearly create heavy traffic:
     - Start a large file download
     - Start a video stream (YouTube, etc.)
     - Run an internet speed test

2. **Start Capture**
   - Open Wireshark and begin capture on active interface.
   - Let it run 20–30 seconds under normal conditions.

3. **Trigger Traffic Spike**
   - Perform the chosen high‑traffic action.
   - Allow it to run for 30–120 seconds.

4. **Stop Capture**
   - Stop the heavy activity.
   - Click **Stop** in Wireshark.

5. **Open I/O Graph**
   - Go to **Statistics → I/O Graph**.
   - Default view shows time on X‑axis and packets or bits per time unit on Y‑axis.

6. **Identify Traffic Spike**
   - Look for a clear "mountain" shape (sudden rise and fall).
   - Note:
     - **Start time** of spike (approximate seconds)
     - **Peak time** and **height** (packets or bits per second)
     - **End time** of spike

7. **Screenshot the Graph**
   - Take a screenshot clearly showing the spike and baseline.
   - Save it for your report.

8. **Filter Packets Around the Spike (Optional Advanced Step)**
   - In main Wireshark window, use a display filter on relative time, e.g.:
     - `frame.time_relative >= X && frame.time_relative <= Y`
   - This keeps only packets during the spike interval.

9. **Analyze Protocols Causing Spike**
   - With spike packets filtered, open **Statistics → Protocol Hierarchy** again.
   - Check which protocols have highest % Bytes during spike.
   - Typically, you’ll see HTTPS/HTTP, TLS, or streaming protocols dominating.

10. **Describe the Spike in Report**

    Include:

    - Time range of spike
    - Peak rate (from I/O graph)
    - Dominant protocol(s) during spike
    - Real‑world action that caused spike (e.g., "starting YouTube video", "downloading ISO file").

### 4.3 Viva Questions

- **Q:** What does the I/O Graph show?
  - **A:** Traffic volume over time (packets or bits per interval); helps spot spikes and anomalies.

- **Q:** How do you link a spike to user activity?
  - **A:** By matching the spike’s timestamp with known user actions (e.g., time when a download or video stream was started).

- **Q:** What could a sudden unexplained spike indicate in a real investigation?
  - **A:** Possible data exfiltration, DDoS participation, or other abnormal activity.

---

## 5. Practical 4 – Packet Capture & Protocol Analysis (HTTP + DNS)

**Objective:** Capture live network traffic, filter for **HTTP and DNS**, extract artifacts (visited sites, DNS queries), export evidence, and reconstruct a **timeline of user activity**.

### 5.1 What You Learn

- How to isolate HTTP and DNS traffic using display filters
- How to extract visited domains and HTTP status codes
- How to view full HTTP conversations using **Follow TCP Stream**
- How to identify DNS queries and responses (including NXDOMAIN)
- How to export HTTP objects and filtered packets as evidence
- How to build a chronological activity timeline

### 5.2 Very Important Concepts for This Practical

#### 5.2.1 HTTP GET Requests

A typical GET request:

```http
GET / HTTP/1.1
Host: neverssl.com
User-Agent: Mozilla/5.0 (...)
```

From this, you can learn:

- **Visited site** – from `Host`
- **Exact resource** – from the path `/...`
- **Browser/OS** – from `User-Agent`

#### 5.2.2 HTTP Status Codes (Common)

- 200 – OK (success)
- 301 – Moved Permanently
- 302 – Temporary Redirect
- 304 – Not Modified (use cached copy)
- 400 – Bad Request
- 401 – Unauthorized
- 403 – Forbidden
- 404 – Not Found
- 500 – Internal Server Error

These codes indicate what happened with each request.

#### 5.2.3 DNS A Record and NXDOMAIN

- **A record** – IPv4 address for a domain.
- **NXDOMAIN** – DNS response indicating the domain does **not** exist.

Example: a query for `test.local` that returns NXDOMAIN shows an attempted but failed lookup.

### 5.3 Capture Phase – Generating Traffic

1. **Install & Launch Wireshark**
   - Ensure WinPcap/Npcap (Windows) or appropriate capture permissions (Linux/macOS).
   - Run Wireshark **as Administrator/root** so it can capture packets.

2. **Select Active Interface**
   - On the main screen, identify the interface (Wi‑Fi/Ethernet) with live growing graph.
   - Double‑click it to start live view.

3. **Start Capture**
   - Click the **blue shark fin**.

4. **Generate Test Traffic**
   - Open a web browser and visit:
     - `http://neverssl.com` (HTTP, unencrypted)
     - One or two more HTTP/HTTPS sites of your choice (e.g., a university site, `https://wikipedia.org` just for comparison).
   - In a new tab, type `test.local` in the address bar and press Enter (to generate a DNS query that will likely fail).

5. **Stop Capture**
   - After ~30–60 seconds, stop capture with the red square.

### 5.4 Extracting HTTP Artifacts (Visited Sites)

1. **Filter for HTTP Traffic**
   - In display filter bar, type:

     ```
     http
     ```

   - Press Enter.

2. **Locate GET Requests**
   - Look at **Info** column for entries like `GET / HTTP/1.1`.
   - Select a GET packet.

3. **View HTTP Headers**
   - In the **Packet Details** pane, expand **Hypertext Transfer Protocol**.
   - Note:
     - `Request Method: GET`
     - `Request URI: /...`
     - `Host: domain.name`
     - Response status (from corresponding response packet): `HTTP/1.1 200 OK` etc.

4. **Fill the HTTP Artifacts Table**

   Example structure for your report:

   | # | Domain (Host header) | HTTP Method | Status Code | Timestamp |
   |---|----------------------|------------|-------------|-----------|
   | 1 | neverssl.com | GET | 200 | [time] |
   | 2 | example.com | GET | 301 | [time] |
   | 3 | ... | GET | 200/404/etc. | [time] |

   - Timestamp can be taken from **Time** column of the GET packet.

### 5.5 Follow HTTP/TCP Stream (Conversation Reconstruction)

1. In filtered HTTP view, right‑click a GET packet.
2. Select **Follow → HTTP Stream** (or **Follow → TCP Stream** if HTTP option not present).
3. A new window appears showing full request/response conversation:
   - Red text = client → server (request)
   - Blue text = server → client (response)
4. Optionally, save or screenshot this view for your report.

### 5.6 Extracting DNS Evidence

1. **Apply DNS Filter**
   - Clear the filter, then type:

     ```
     dns
     ```

   - Press Enter.

2. **Find `test.local` Query**
   - Look in **Info** column for "Standard query" mentioning `test.local`.
   - Select that packet.

3. **Inspect DNS Query Details**
   - In Packet Details, expand **Domain Name System (query)**.
   - Under **Queries**, note:
     - `Name: test.local`
     - `Type: A` (IPv4 address lookup)

4. **Check Response**
   - Look for subsequent DNS packet with Info like "Standard query response" for `test.local`.
   - Often this is `NXDOMAIN` (no such domain) or simply no answer.

5. **Fill DNS Query Evidence Table**

   Example:

   | Query Name | Query Type | Response (if any) | Packet # |
   |------------|-----------|-------------------|----------|
   | test.local | A | No response / NXDOMAIN | [packet no.] |

   - Packet number comes from **No.** column in Wireshark.

### 5.7 Exporting Evidence

#### 5.7.1 Export HTTP Objects (Files)

1. Go to **File → Export Objects → HTTP**.
2. Review the list of HTTP objects (HTML pages, images, etc.).
3. Select interesting entries (or **Export All**).
4. Save them to a folder.

You now have local copies of files the user’s browser downloaded during capture.

#### 5.7.2 Export DNS Packets as `dns_evidence.pcapng`

1. While DNS filter (`dns`) is active, go to **File → Export Specified Packets**.
2. Choose to export **All packets** (or only selected range if instructed).
3. Ensure format is **pcapng**.
4. Name file `dns_evidence.pcapng` and save.

### 5.8 Timeline Reconstruction

Goal: Show the **sequence** of events – from DNS query to HTTP response.

1. Clear any display filter to see all packets.
2. Ensure the **Time** column is visible.
3. Identify and document key events in order:

   - DNS query for a domain
   - DNS response
   - TCP 3‑way handshake (SYN, SYN‑ACK, ACK)
   - HTTP GET request
   - HTTP response (status code, e.g., 200/404)

4. Create a narrative timeline in your report:

   ```text
   Time 00:02.100 – DNS query A for neverssl.com
   Time 00:02.150 – DNS response: neverssl.com → 45.79.xx.xx
   Time 00:02.300 – TCP SYN from client to 45.79.xx.xx:80
   Time 00:02.310 – TCP SYN‑ACK from server
   Time 00:02.320 – TCP ACK from client (connection established)
   Time 00:02.400 – HTTP GET / HTTP/1.1 (Host: neverssl.com)
   Time 00:02.500 – HTTP/1.1 200 OK (HTML page delivered)
   ```

5. Add brief forensic comments, e.g., "User attempted to visit neverssl.com and successfully loaded the main page".

### 5.9 Suggested Report Structure for Practical 4

1. **Captured Interface**
   - Interface name
   - Capture duration (seconds)
   - Total packets captured

2. **HTTP Artifacts (Visited Sites via GET)**
   - Table with Domain (Host), Method, Status, Timestamp.

3. **DNS Query Evidence**
   - Table with Query name, Type, Response, Packet number.

4. **Screenshots/Evidence**
   - HTTP filter view with GET requests.
   - Follow TCP/HTTP Stream window.
   - Export Objects window.

5. **Timeline Reconstruction**
   - Text timeline with DNS → TCP → HTTP sequence.

6. **Forensic Analysis & Conclusion**
   - List visited sites.
   - Explain what DNS queries show about user intent.
   - Note any failed lookups or 404/500 errors.
   - Comment on importance of Wireshark for reconstructing activity even if browser history is cleared.

### 5.10 Important Viva Questions (with Short Answers)

- **Q:** Why was `neverssl.com` chosen for this lab?
  - **A:** It uses plain HTTP (no encryption), so headers and content are fully visible for analysis.

- **Q:** What can the HTTP `Host` header tell you?
  - **A:** The exact domain the browser requested, revealing visited websites.

- **Q:** What does a DNS NXDOMAIN response mean?
  - **A:** The domain does not exist – the lookup failed.

- **Q:** How does `Follow TCP Stream` help in investigations?
  - **A:** It reconstructs the full conversation between client and server in correct order.

- **Q:** Is packet capture legal everywhere?
  - **A:** No. You must have authorization (own system or explicit permission). In real cases, often a warrant or written consent is required.

---

## 6. General Tips for All Four Practicals

### 6.1 Good Laboratory Practice

- Always **note interface name**, **start time**, **end time**, and **capture duration**.
- Generate sufficient traffic (especially web browsing/streaming) during captures.
- Use **display filters** (http, dns, tcp, ip.addr == x.x.x.x, etc.) instead of scrolling blindly.
- Take **clear screenshots** of key windows (Protocol Hierarchy, Endpoints, Conversations, I/O Graph, Follow Stream, Export Objects).

### 6.2 Writing Strong Practical Answers

For each practical, include sections:

1. **Objective** – 2–3 lines explaining purpose.
2. **Tools Used** – Wireshark version, OS, browser.
3. **Procedure** – Numbered steps actually followed.
4. **Observations** – Tables, screenshots, and values.
5. **Analysis** – Interpretation: what do the results mean?
6. **Conclusion** – Short summary linking result back to objective.

### 6.3 Viva Preparation Checklist

Be ready to explain:

- What Wireshark is and how it works (sniffer + analyzer).
- Difference between **capture filters** and **display filters**.
- Basic properties of TCP, UDP, HTTP, HTTPS, DNS.
- Common port numbers and associated services.
- Why HTTP traffic is easier to analyze than HTTPS.
- How to use: Protocol Hierarchy, Endpoints, Conversations, I/O Graph, Follow Stream.

If you can confidently speak through the steps and reasoning in this file, you will handle both the **practical write‑up** and the **viva** strongly.
