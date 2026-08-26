# Genome Assembly Research Network 🧬💻

An enterprise-grade, secure, and fault-tolerant network designed and simulated in Cisco Packet Tracer. This project serves as the final group capstone for the **Computer Network (CSE421)** course.

---

## 👥 Team Members & Contributions
*   **MD. Aminul Islam Ornab (ID: 22101569)** - *Lead Network Architect*  

*   **Afaf Binta Shamsuddin (ID: 22101844)** - *Systems & Services Engineer*  

*   **Dipannita Banik (ID: 24301043)** - *Routing & Validation Analyst*  

*   **Syed Isfar Rahman (ID: 22201312)** - *Technical Writer & Quality Lead*  


---

## 📌 Project Overview
A national bioinformatics lab is working on a large genome assembly project. The operation is distributed across six specialized research units. This project implements a fully routed network topology enabling seamless raw biological data sharing, centralized web resource access, and secure inter-unit email communication.

### The Six Research Units:
1.  **Sequencing Core Lab (SCL) [520 Hosts]:** Generates raw DNA reads (FASTQ files). Hosts central DNS, Web, Email, and DHCP servers.
2.  **Read Mapping Unit (RMU) [300 Hosts]:** Aligns raw reads to reference genomes.
3.  **De Bruijn Graph Unit (DBG) [220 Hosts]:** Builds overlapping k-mer graphs for *de novo* assembly.
4.  **Motif Discovery Unit (MDU) [160 Hosts]:** Searches regulatory genomic patterns. Hosts local DHCP and Email servers.
5.  **Genome Validation Unit (GVU) [90 Hosts]:** Conducts sequence quality checks and validation.
6.  **Archive Storage Node (ASN) [45 Hosts]:** The final static data repository (configured strictly with static IPs).

---

## 🛠️ Network Architecture & Design

### 1. VLSM Addressing Design
The base network block was dynamically derived from the student IDs of the first two team members:
*   First Octet = **69** (Ornab: `22101569`)
*   Second Octet = **44** (Afaf: `22101844`)
*   Base IPv4 Subnet: **`69.44.0.0/16`**

Subnets were calculated hierarchically using **Variable Length Subnet Masking (VLSM)**, sorted descendingly from the largest host requirement to the smallest point-to-point links to optimize address space and reduce IP wastage.

### 2. Hybrid DHCP Setup
*   **Server-Based Pools:** Managed by dedicated physical servers in SCL (for SCL LAN and GVU LAN via DHCP Relay) and MDU (for MDU LAN). Static IP exclusions protect critical gateway, server, and printer IPs.
*   **Router-Based Pools:** Run natively on the RMU and DBG routers. Dynamic scopes implement host offset exclusions starting exactly at **`.69`** (RMU) and **`.44`** (DBG) to match team student ID constraints.
*   **DHCP Relay Agents:** Configured with `ip helper-address 69.44.0.5` on SCL and GVU routers, converting local DHCP broadcast requests into routed unicast requests destined for SCL's central server.

### 3. Routing Matrix
*   **RIPv2 Dynamic Routing:** Runs across SCL, RMU, DBG, MDU, and GVU routers using classless updates (`no auto-summary`). Security is reinforced using `passive-interface` on all LAN gateways.
*   **Next-Hop Static Route:** Configured on SCL to reach the ASN LAN.
*   **Exit-Interface Default Static Route:** Configured on the stub ASN router pointing to `Serial0/2/0` toward GVU.
*   **Recursive Static Route:** Configured on RMU to reach MDU’s LAN via its shared backbone IP (`69.44.8.195`).
*   **Static Redistribution:** Configured on GVU using the `redistribute static` command, advertising the non-RIP ASN LAN route dynamically to all RIPv2 neighbors.
*   **Floating Static Route (Failover):** Configured on DBG pointing to SCL's LAN via MDU with an **Administrative Distance (AD) of 123**. This backup path remains dormant unless the primary RIPv2 (AD 120) path fails.

### 4. Application Layer Services
*   **Web Portal:** Hosted on SCL Web Server (`69.44.0.3`), reachable from all networks at **`www.genome-lab.bio`**.
*   **Central DNS Server:** Located at `69.44.0.2` mapping host A-records for web and mail servers.
*   **Cross-Domain Email:** Connects the **`mail.sequence.bio`** (SCL) and **`mail.motif.bio`** (MDU) domains using SMTP (TCP 25) and POP3 (TCP 110).

---

## 📊 Detailed IP Addressing Table

| Unit / Link Name | Required Hosts | Subnet Mask | CIDR | Network Address | Usable IP Range |
| :--- | :---: | :--- | :--- | :--- | :--- |
| **Sequencing Core Lab (SCL)** | 520 | `255.255.252.0` | `/22` | `69.44.0.0` | `69.44.0.1 - 69.44.3.254` |
| **Read Mapping Unit (RMU)** | 300 | `255.255.254.0` | `/23` | `69.44.4.0` | `69.44.4.1 - 69.44.5.254` |
| **De Bruijn Graph Unit (DBG)**| 220 | `255.255.255.0` | `/24` | `69.44.6.0` | `69.44.6.1 - 69.44.6.254` |
| **Motif Discovery Unit (MDU)**| 160 | `255.255.255.0` | `/24` | `69.44.7.0` | `69.44.7.1 - 69.44.7.254` |
| **Genome Validation (GVU)** | 90 | `255.255.255.128`| `/25` | `69.44.8.0` | `69.44.8.1 - 69.44.8.126` |
| **Archive Storage Node (ASN)**| 45 | `255.255.255.192`| `/26` | `69.44.8.128` | `69.44.8.129 - 69.44.8.190` |
| **Backbone Switch (Shared)**  | 3 | `255.255.255.248`| `/29` | `69.44.8.192` | `69.44.8.193 - 69.44.8.198` |
| **Point-to-Point WAN Links**  | 2 each| `255.255.255.252`| `/30` | `69.44.8.200 - .220` | *Refer to project documentation* |

---

## 📂 Repository Structure

```text
├── CSE421_Project.pkt          # Main Cisco Packet Tracer Project Simulation File
├── Router_Configs/             # Text files containing full router CLI configs
│   ├── SCL_Router_Config.txt
│   ├── RMU_Router_Config.txt
│   ├── DBG_Router_Config.txt
│   ├── MDU_Router_Config.txt
│   ├── GVU_Router_Config.txt
│   └── ASN_Router_Config.txt
├── Screenshots/                # System verification screenshots (Web, Mail, Pings)
└── Documentation/              # Core documentation files & VLSM Calculations
