# 🛡️ Enterprise SOC Lab Simulation & Threat Detection

![Status: In Progress](https://img.shields.io/badge/Status-In%20Progress-blue)
![Role: SOC Analyst](https://img.shields.io/badge/Role-SOC%20Analyst%20%2F%20Security%20Engineer-success)
![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-00AEEF?logo=wazuh&logoColor=white)
![pfSense](https://img.shields.io/badge/pfSense-Firewall-black?logo=pfsense&logoColor=white)

## 📌 Project Overview
This project is a miniaturized, yet fully functional, Enterprise Security Operations Center (SOC) lab. It is designed to simulate a real-world corporate network, focusing on **Network Segmentation**, **Centralized Log Management**, and **Advanced Threat Detection**. 

The primary objective of this lab is not just to collect logs, but to engineer high-fidelity correlation rules that map to the **MITRE ATT&CK** framework, actively reducing 'Alert Fatigue' and highlighting critical security incidents.

## 🏗️ Network Architecture
<img width="866" height="735" alt="image" src="https://github.com/user-attachments/assets/7fb3de3e-10ca-4c06-ba11-eec89f177209" />

The lab environment is built on virtualized infrastructure with strict segmentation:
* **pfSense (Perimeter Firewall/Router):** Manages traffic routing between external networks, the DMZ, and the internal LAN.
* **DMZ (Demilitarized Zone):** Hosts vulnerable Web Servers (Windows/XAMPP running DVWA) simulating public-facing assets.
* **Internal LAN:** Houses the Wazuh SIEM Manager and internal monitoring tools.
* **Attacker Machine:** Kali Linux operating from an external subnet.

## 🛠️ Technology Stack
* **SIEM:** Wazuh
* **NIDS/IPS:** Suricata
* **Firewall/Routing:** pfSense
* **Log Ingestion:** Rsyslog, Wazuh Agent
* **Target Environment:** Windows Server, Ubuntu Linux, Apache (XAMPP)
* **Offensive Tools:** Hydra, Nmap, Burp Suite
