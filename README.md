# Day 27: Target Reconnaissance & Service Discovery (OWASP BWA)

## 📌 Project Overview
This repository contains the technical walkthroughs, command logs, discovery footprints, and consolidated summaries compiled during **Day 27 of my Vulnerability Assessment and Penetration Testing (VAPT) Internship with TriosCyber** (in partnership with Ernith).

The primary objective of this project was to perform full-spectrum active reconnaissance against an authorized local lab host. The execution workflow progressed from layer-3 network path validation to full-port service discovery via **Nmap**, web application software fingerprinting via **WhatWeb**, and fast directory brute-forcing via **FFUF** to build a comprehensive attack surface map.

---

## 🛠️ Lab Environment & Target Details
* **Target Host IP Address:** `192.168.145.134`
* **Target Operating Environment:** OWASP Broken Web Applications VM (`owaspbwa`)
* **Scope Bounds:** Authorized Internal Lab Virtual Network Segment (`192.168.145.0/24`)
* **Primary Tools Utilized:** `ping`, `nmap`, `whatweb`, `ffuf`

---

## 🚀 Step-by-Step Test Sequence & Technical Findings

### 1. Connectivity & Latency Verification
Issued ICMP echo request packets to verify network reachability, calculate average round-trip time (RTT) latencies, and check connection stability before starting heavy multi-threaded scans.

```bash
ping -c 4 192.168.145.134
```

* **Observed Metrics:** 4 packets transmitted, 4 received (0% packet loss). 
* **Latency Profile:** min/avg/max/mdev = 0.702/0.967/1.368/0.252 ms.
* **Triage Analysis:** Confirmed target host is active, stable, and highly responsive.

---

### 2. Nmap Service & Version Discovery Scan
Executed an optimized full-range TCP version detection scan (`-sV`), paired with default Nmap Scripting Engine queries (`-sC`) and aggressive scan pacing adjustments (`-T4`) across all port structures:

```bash
nmap -sC -sV -p- -T4 192.168.145.134
```


#### 📋 Discovered Open Ports & Banner Metadata:
* **`22/TCP` (Open):** `SSH` → OpenSSH 5.3p1 Debian 3ubuntu4 (Ubuntu Linux).
* **`80/TCP` (Open):** `HTTP` → Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2).
* **`139/TCP` (Open):** `netbios-ssn` → Samba smbd 3.X - 4.X (workgroup: WORKGROUP).
* **`143/TCP` (Open):** `imap` → Courier Imapd (released 2008).
* **`443/TCP` (Open):** `ssl/http` → Apache httpd 2.2.14 (OpenSSL/0.9.8k).
* **`445/TCP` (Open):** `microsoft-ds` → Samba smbd 3.X - 4.X (workgroup: WORKGROUP).
* **`5001/TCP` (Open):** `java-object` → Java Object Serialization framework listener.
* **`8081/TCP` (Open):** `http` → Jetty container engine version 6.1.25.
* **`8088/TCP` (Open):** `http` → Apache Tomcat/Coyote JSP engine version 1.1.

---

### 3. Web Application Stack Fingerprinting
Ran the `WhatWeb` diagnostic engine against the default port 80 web path to automatically pull application configurations, component variations, and response parameters:

```bash
whatweb http://192.168.145.134
```

* **HTTP Transaction Status:** `200 OK`.
* **Web Server Daemon Platform:** `Apache/2.2.14 (Ubuntu)`.
* **Backend Engines:** `PHP/5.3.2-1ubuntu4.30 with Suhosin-Patch`, `Python/2.6.5`, `Perl/v5.10.1`, `Ruby (Passenger/4.0.38)`.
* **Cryptographic SSL Library:** `OpenSSL/0.9.8k`.
* **Client Frontend Frameworks:** `jQuery/1.3.2`.

---

### 4. Directory & Endpoint Discovery
Conducted fast HTTP directory brute-forcing using `ffuf` to locate unlinked endpoints, system backups, and hidden deployment directories:

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://192.168.145/FUZZ -mc 200,301,302
```


#### 📋 Discovered Paths & Operational Descriptions:
* **`/.bash_history` (`200 OK` | 302 B):** Leaked administrative terminal shell logging histories.
* **`/cgi-bin/` (`200 OK` | 1,070 B):** Server-side CGI executable scripting repository paths.
* **`/phpmyadmin` (`301 Moved` | 242 B):** Web forwarding index for the central MySQL database management system.
* **`/assets, /images, /javascript` (`301 Moved`):** Core application static resource folder routes.
* **`/joomla, /phpBB2, /test` (`301 Moved`):** Sub-hosted web applications active on the host root directory.

---

## 📊 Consolidated Reconnaissance Summary Matrix

| Assessment Phase | Tooling Engine | Primary Operational Objective | Core Technical Findings & Target Metrics |
| :--- | :--- | :--- | :--- |
| **Connectivity Tracking** | `ping` | Evaluate network latency and confirm host reachability state. | Target host alive; zero packet drops recorded. Average RTT latency: `0.967ms`. |
| **Port & Service Scan** | `nmap` | Map full open port configurations and daemon version signatures. | Identified 9 open services spanning SSH, HTTP/S, Samba, IMAP, Java, Tomcat, and Jetty. |
| **Web Stack Review** | `whatweb` | Identify response header components and backend platform code. | Outdated software builds exposed: Apache 2.2.14, PHP 5.3.2, OpenSSL 0.9.8k, Ubuntu Linux. |
| **Directory Discovery** | `ffuf` | Expose hidden system paths via wordlist directory brute-forcing. | Leaked critical information endpoints: raw `/.bash_history`, `/phpmyadmin`, and hosted web frameworks. |
| **Triage Status** | **COMPLETED** | Consolidate and structure collected target intelligence vectors. | Completed target baseline map; successfully prepped the target for vulnerability scoring assessments. |

---

## 🛡️ Key Takeaways & Remediation Guidelines

### 1. Restrict and Purge Sensitive Informational Disclosures
Web roots and operational staging directories must be swept to purge residual development properties or local terminal traces, such as the exposed `/.bash_history` file. Leaving shell history tables visible enables unauthenticated malicious actors to easily extract administrative system inputs, variable setups, and path parameters.

### 2. Suppress Verbose Web Server Response Banners
Reconfigure internal server header configurations to mitigate banner-grabbing profiling routines. Adjust active Apache environment properties to enforce strict production-level parameter constraints, which completely masks underlying sub-module software and operating system builds:
```apache
ServerTokens Prod
ServerSignature Off
```


### 3. Fortify Access Restrictions on Administrative Portals
Isolate management application paths such as `/phpmyadmin` from direct internet visibility. Ensure these endpoints are placed behind strict IP-based access white-lists or mandate robust corporate VPN connectivity alongside Multi-Factor Authentication (MFA) to prevent unauthorized credential brute-forcing.

---

## 📂 Repository Structure
```plaintext
.
├── README.md
├── screenshots/
│   ├── icmp_ping_connectivity.png
│   ├── nmap_service_discovery.png
│   ├── whatweb_service_discovery.png
│   └── ffuf_directory_fuzzing.png
└── outputs/
    ├── nmap_full_scan_report.txt
    └── ffuf_discovered_endpoints.txt
```

---

## 👤 Author
**Azeez Umar Opeyemi**
* 💼 **Role:** VAPT Intern at TriosCyber
* 📧 **Email:** umaropeyemiazeez@gmail.com
* 🐙 **GitHub:** [itzumaz](https://github.com/itzumaz)
* 🔗 **LinkedIn:** [Azeez Umar Opeyemi](https://www.linkedin.com/in/azeez-umar-opeyemi-201a433a4/)
