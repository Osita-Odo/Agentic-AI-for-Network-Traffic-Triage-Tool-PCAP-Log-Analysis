# PCAP Log Analysis with a Browser-Based Network Traffic Triage Tool

**Lab report | Osita Kingsley Odo | October 2026**

## Table of Contents

1. [Objective](#1-objective)
2. [Environment and Tools](#2-environment-and-tools)
3. [Capturing the Traffic](#3-capturing-the-traffic)
4. [Exporting and Loading the Capture](#4-exporting-and-loading-the-capture)
5. [Findings](#5-findings)
6. [Improvements Made After the First Test](#6-improvements-made-after-the-first-test)
7. [Lessons Learnt](#7-lessons-learnt)

---

## 1. Objective

The aim of this lab was to capture real network traffic in a controlled environment, analyse it with a triage tool I designed, and produce a shareable report that a SOC analyst could use as a starting point for investigation. The lab also tested how the tool handles privacy, since packet captures contain IP addresses that should not always appear in reports.

## 2. Environment and Tools

| Component | Details |
|---|---|
| **Capture machine** | Kali Linux virtual machine, capturing on interface eth0 |
| **Capture tool** | Wireshark |
| **Traffic source** | Browsing to terracybconsulting.com, a company website I built and own, so that all captured traffic came from a system I am authorised to monitor |
| **Analysis tool** | Network Traffic Triage Tool, a browser-based application I built in VS Code with the Cline AI coding agent. It reads a Wireshark CSV export, runs a set of detection rules and explains each alert in plain language. All processing happens locally in the browser and no data is uploaded to a server or external API. |

**Prompt used to build the tool:** [Here](https://docs.google.com/document/d/1fCtTTVzvLgUYv_zbPKF5kIoNgpmJTUh41DKzyyhB3MY/edit?tab=t.vser1gdcw109)

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/65514ccc-bf07-4c4f-9104-5b7610dab82c" />
*Figure 1. Building the triage tool in VS Code with Cline. The automated test suite passed 98 tests with no failures.*

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/97e2c601-b7c2-435a-ac11-638ad13a8a79" />
*Figure 2. The tool's landing page, with the local processing notice and the CSV upload area.*

## 3. Capturing the Traffic

In Kali Linux, I opened Wireshark and started a capture on eth0. I then opened Firefox and visited terracybconsulting.com to generate web traffic while the capture was running.

<img width="941" height="646" alt="image" src="https://github.com/user-attachments/assets/d679fc79-715d-41d9-a422-42fc41a8e412" />
*Figure 3. The TerraCyber Consulting website loaded in Firefox on Kali Linux during the capture.*

The capture recorded a mix of TCP, TLS 1.2 and TLS 1.3 traffic between the Kali host (192.168.10.5) and several external servers on port 443, along with DNS and ARP traffic.

<img width="965" height="586" alt="image" src="https://github.com/user-attachments/assets/fb50e8e8-c2a5-494b-97d6-fc25f2b758a5" />
*Figure 4. The completed capture in Wireshark.*

## 4. Exporting and Loading the Capture

I saved the capture in my Kali home directory and exported the packet list as a CSV file (**File > Export Packet Dissections > As CSV**), which is the format the triage tool accepts. I sent the file to my Gmail account and downloaded it on my host machine, then loaded it into the tool. The tool parsed **8,107 packets** across 7 columns with no missing fields.

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/e0200548-1987-4712-b23d-4afa0b96717b" />
*Figure 5. File details after upload: 8,107 packets parsed successfully.*

## 5. Findings

### 5.1 Protocol distribution

| Protocol | Packets | Share |
|---|---:|---:|
| TCP | 1,870 | 23.1% |
| DNS | 406 | 5.0% |
| ARP | 4 | 0.0% |
| HTTP | 4 | 0.0% |
| Other | 5,823 | 71.8% |

The large "Other" category is most likely encrypted web traffic (TLS), which Wireshark labels separately and which the current version of the tool does not yet classify on its own. This is expected for HTTPS browsing.

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/7d1c420f-2bdb-4d49-b114-0ed5db3ba8c2" />
*Figure 6. Protocol distribution and filter options.*

### 5.2 Alerts

The tool raised **59 alerts**: 7 High, 16 Medium, 15 Low and 21 Informational. Each alert explains why the activity could matter, lists possible benign explanations and suggests next investigation steps.

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/eabe141f-f408-4199-9361-98c3599902e2" />
*Figure 7. Alert summary by severity.*

The High alerts were driven mainly by packet volume. For example, one source host sent 1,279 packets against a configured threshold of 50, and rule R7 (TCP SYN-heavy activity) triggered because 192.168.10.5 sent 53 SYN packets against a threshold of 30.

<img width="940" height="510" alt="image" src="https://github.com/user-attachments/assets/57ef43a1-3f80-478a-b285-dbae51bcc308" />
*Figure 8. A High Activity Host alert. The source host has been redacted.*

### 5.3 Interpretation

Because the capture consisted of ordinary browsing to my own website, these alerts are consistent with normal activity. A modern website opens many parallel HTTPS connections to content delivery networks and third-party services, which produces high packet counts and many SYN packets in a short time. The result shows that the tool's thresholds are set low for teaching purposes and would need to be compared against a normal baseline before being used to judge real traffic.

## 6. Improvements Made After the First Test

### 6.1 Hiding IP addresses

After the first test, I noticed that real source and destination IP addresses appeared throughout the results and would be carried into any exported report. This is a confidentiality risk when reports are shared. I added a **Hide IP addresses** option that replaces each address with a consistent alias (for example, HOST-04), so hosts can still be tracked across the report without revealing the underlying addresses.

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/17cb8f5a-3066-4e2f-97ea-4f7cb9dfdfc9" />
*Figure 9. The Hide IP addresses option at the top of the tool, switched on.*

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/520e4909-c96b-4374-98bb-5e1318342470" />
*Figure 10. Top source and destination hosts shown as aliases with Hide IP addresses switched on.*

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/3e675e9e-6131-4ba0-a8d7-254416c617fb" />
*Figure 11. Alerts displayed with host aliases, alongside the updated project in VS Code.*

### 6.2 Report formats and severity selection

I also added export options so the report can be generated as PDF, Word, HTML, CSV or Markdown. In addition, I added checkboxes to choose which alert severities appear in the report: High, Medium, Low, Informational or all of them. In the example below, only the 7 High alerts are selected. The investigation score and rule status still cover the full analysis whatever is selected.

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/3b4d3661-5b01-4042-a3d0-4e68ce266cc6" />
*Figure 12. Export options with severity selection set to High only.*

## 7. Lessons Learnt

- **Detection thresholds need context.** Volume-based rules flagged normal browsing as High severity, which shows why alerts should be read as prompts to investigate rather than proof of malicious activity.
- **Privacy should be built in from the start.** Aliasing IP addresses before export makes reports safer to share with people outside the investigation.

---

> **Disclaimer:** This is an educational lab. All traffic was captured from systems I own.
