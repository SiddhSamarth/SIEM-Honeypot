# Cloud Threat Telemetry & SIEM Honeypot Geolocation Dashboard

Deploying an exposed Azure Windows virtual machine honeypot to capture RDP brute-force telemetry (Event ID 4625) and visualize global attack origin distribution using Microsoft Sentinel Workbooks and KQL.

---

## Scope & Lab Environment

* **Target Host:** Azure Windows Virtual Machine with RDP (port 3389) exposed to the public Internet and host firewall disabled.
* **Telemetry Collected:** Windows Security Event Log (Event ID 4625 - An account failed to log on).
* **Enrichment:** PowerShell script harvesting failed logons and querying `ipgeolocation.io` API for latitude, longitude, and country metadata.
* **Ingestion & Visualization:** Custom Log Analytics table (`FAILED_RDP_WITH_GEO_CL`) aggregated via KQL and plotted in a Microsoft Sentinel Map Workbook.
* **Exercise Provenance:** This repository documents a hands-on cloud honeypot deployment exercise. The honeypot architecture, PowerShell extraction methodology, and visual walkthrough material are adapted from lab coursework by Larry ([@laaaaaarry](https://github.com/laaaaaarry/SIEM-Honeypot)). Configuration notes and KQL queries are documented below for technical reference.

---

## Architecture & Data Pipeline

```
[ Public Internet Attackers (Port 3389 / RDP) ]
                       │
                       ▼
┌──────────────────────────────────────────────┐
│       Azure Windows VM Honeypot              │
│  - Windows Defender Firewall: OFF            │
│  - NSG Inbound: Any/Any                      │
│  - Security.evtx: Event ID 4625 (Failed RDP) │
│  - PowerShell Log Harvester (ipgeolocation)  │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│       Azure Log Analytics Workspace          │
│  - Custom Log Table: FAILED_RDP_WITH_GEO_CL  │
│  - Extracted Fields: Lat, Lon, Country, User │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│       Microsoft Sentinel (Cloud SIEM)        │
│  - KQL Aggregation Query                     │
│  - Global Threat Map Workbook Visualization  │
└──────────────────────────────────────────────┘
```

---

## Detailed Implementation Steps

### 1. Honeypot Virtual Machine Deployment
Deploy a Windows 10/Server VM in Azure. Configure an unrestricted Network Security Group (NSG) rule (`Priority 100`, Inbound `Any`, Destination Port `*`, Action `Allow`) to maximize attack surface visibility.

<p align="center">
  <img src="./images/Screenshot_2024-03-04_234052_1.png" alt="Azure VM Creation" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-04_234951_2.png" alt="NSG Allow All Inbound" width="750" />
</p>

---

### 2. Log Analytics Workspace & Defender for Cloud Configuration
Provision an Azure Log Analytics Workspace (`LA-Honeypot`) and configure Microsoft Defender for Cloud to collect **All Events** from connected virtual machines.

<p align="center">
  <img src="./images/Screenshot_2024-03-04_235211_3.png" alt="Log Analytics Provisioning" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_001507_4.png" alt="Defender for Cloud Data Collection" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_001441_5.png" alt="All Events Selection" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_001850_6.png" alt="Connecting VM to Workspace" width="750" />
</p>

---

### 3. Sentinel Ingestion & Host Firewall Deactivation
Enable Microsoft Sentinel on top of the workspace. Connect to the honeypot via Remote Desktop Protocol (RDP) using its public IP address, and disable all three Windows Defender Firewall profiles (Domain, Private, Public) to ensure ICMP and TCP probes reach the OS without restriction.

<p align="center">
  <img src="./images/Screenshot_2024-03-05_002034_7.png" alt="Sentinel Creation" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_002503_8.png" alt="RDP Connection" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_002625_9.png" alt="Logging in to VM" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_003355_10.png" alt="Disabling Host Firewall" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_003517_11.png" alt="Firewall State: Disabled" width="750" />
</p>

---

### 4. PowerShell Telemetry Harvester & IP Geolocation Enrichment
A custom PowerShell script runs continuously in the VM background. It queries Windows Security Event Log for **Event ID 4625** (failed logon attempts), parses the attacker's source IP, queries the `ipgeolocation.io` API, and outputs structured records to `C:\ProgramData\failed_rdp.log`.

<p align="center">
  <img src="./images/Screenshot_2024-03-05_004203_12.png" alt="PowerShell Harvester Script" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_004906_13.png" alt="PowerShell Execution Output" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_005801_14.png" alt="Custom Log Output Format" width="750" />
</p>

---

### 5. Custom Table Creation & Field Extraction in Log Analytics
Configure a custom MMA-based log collection table in Log Analytics pointing to `C:\ProgramData\failed_rdp.log` on the honeypot host. Extract raw string telemetry into queryable fields (`latitude_CF`, `longitude_CF`, `country_CF`, `username_CF`, `sourcehost_CF`).

<p align="center">
  <img src="./images/Screenshot_2024-03-05_011352_15.png" alt="Custom Table Creation" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_044726_16.png" alt="Extracting Fields from RawData" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_044921_17.png" alt="Numeric Field Typing" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_045038_18.png" alt="Sample Extraction Verification" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_045412_19.png" alt="Querying Custom Log Table" width="750" />
</p>

---

### 6. Sentinel Workbook World Map Visualization
In Microsoft Sentinel, create a custom Workbook using the **Map** visualizer to project aggregated attack origins geographically.

<p align="center">
  <img src="./images/Screenshot_2024-03-05_045852_20.png" alt="Sentinel Workbook Creation" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_051215_21.png" alt="Map Query Configuration" width="750" />
</p>

<p align="center">
  <img src="./images/Screenshot_2024-03-05_051113_22.png" alt="Global Threat Visualization Map" width="750" />
</p>

#### Extracted KQL Map Aggregation Query:
```kql
FAILED_RDP_WITH_GEO_CL
| where latitude_CF != "" and destinationhost_CF != "samplehost"
| extend Latitude = toreal(latitude_CF), Longitude = toreal(longitude_CF)
| summarize EventCount = count() by Latitude, Longitude, country_CF, label_CF, destinationhost_CF
| project Latitude, Longitude, EventCount, country_CF, label_CF, destinationhost_CF
```

---

## Technical Notes & Operational Limitations

* **PowerShell Harvester Execution:** The log-forwarding script was executed interactively within Windows PowerShell ISE on the target VM during the lab session to process `Security.evtx` records, rather than tracked as a version-controlled repository artifact.
* **Cost & Safety Controls:** In real-world enterprise deployments, honeypots must be deployed in fully segregated subscriptions or VPCs with zero network routes back into corporate internal subnets.

---

## Attribution & Acknowledgments

* Honeypot setup architecture, PowerShell extraction methodology, and lab walkthrough adapted from original exercises by Larry ([@laaaaaarry](https://github.com/laaaaaarry)).

---

## Author & Links

* **Author:** Siddh Samarth
* **GitHub:** [@SiddhSamarth](https://github.com/SiddhSamarth)
* **Portfolio:** [siddhsamarth.in](https://siddhsamarth.in)
* **LinkedIn:** [samarthsiddh](https://www.linkedin.com/in/siddhsamarth/)
