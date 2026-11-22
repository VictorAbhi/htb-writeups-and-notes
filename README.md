# htb-writeups-and-notes
Offensive security write-ups, methodology notes, and defensive remediation guides from Hack The Box (HTB) labs. Focused on API security, Python web apps, Docker/Kubernetes container breakouts, AWS cloud misconfigurations, Active Directory and some defending concepts  specially bridging traditional AppSec with Cloud &amp; AI infrastructure security.

Hack The Box — Offensive/Defensive Security Write-Ups & Cloud/AI Defense Notes

Welcome to my cybersecurity lab repository! This repo documents my hands-on exploitation methodologies, privilege escalation paths, and **blue-team remediation strategies** across retired **Hack The Box (HTB)** machines and Sherlocks.

As an MSc Cybersecurity student (IMT Atlantique) with an **AWS Solutions Architect** and **AWS Machine Learning Engineering** background, I curate and analyze these labs through the lens of **securing modern Cloud, API, Container, and AI/ML infrastructures**.

> **HTB Policy Compliance:** In accordance with Hack The Box rules, this repository **only** contains write-ups for **retired** machines and challenges. No active machines, Pro Lab flags, or active challenge solutions are published here.

---

## Focus Areas
* **Application Security:** SSRF, SSTI (Flask/Jinja2), JWT manipulation, authentication bypasses, and insecure deserialization.
* **Cloud & Container Security:** AWS S3/IAM/LocalStack misconfigurations, Docker container escapes, and Kubernetes cluster enumeration/privilege escalation.
* **Enterprise & Active Directory:** Lateral movement, network pivoting (Ligolo-ng / Chisel), Kerberoasting, and AD privilege escalation.
* **Defensive Engineering & Detection:** Every write-up concludes with actionable architectural fixes, log detection queries, and threat modeling notes relevant to cloud-native and AI workloads.

---

## Repository Structure

```text
htb-writeups-and-notes/
├── 01--Red-Team/          # Web APIs, SSRF, SSTI, Python/Flask flaws, Docker misconfig, AWS
├── 02-Blue-Team-Sherlocks/      # DFIR, log analysis, PCAP & incident response
└── Cheatsheets/                 # Enumeration, Ligolo-ng tunneling, and PrivEsc notes
```

---

## Completed Machines & Labs

| Machine / Lab | Category | Difficulty | Key Vulnerabilities & Takeaways |
| :--- | :--- | :--- | :--- |
| **[Editorial](./01--Application-security/Editorial.md)** | Python & APIs | Easy | API Enumeration, SSRF, Credential Exposure, Git History Analysis |
| *More coming soon...* | — | — | — |

---

## Core Toolkit
* **Recon & Web/API:** `nmap`, `ffuf`, `gobuster`, `Burp Suite`, `curl`, `jq`
* **Cloud & Containers:** `aws-cli`, `kubectl`, `trivy`, `linpeas` / `winpeas`
* **Active Directory & Pivoting:** `BloodHound`, `Impacket`, `NetExec`, `Ligolo-ng`, `Chisel`

---

## 🤝 Connect
* **Author:** Abhishek Adhikari — MSc Information Technology, Cybersecurity Student @ IMT Atlantique (Rennes, France)
* **LinkedIn:** [://linkedin.com](https://www.linkedin.com/in/abhishek-adhikari-8728422a6/)
