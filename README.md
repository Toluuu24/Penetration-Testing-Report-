# Penetration Test Report — Sweeny Barbers Internal Network Assessment

**Prepared by:** University of Plymouth Security (UPS)  
**Client:** Sweeny Barbers  
**Classification:** Confidential  
**Methodology:** PTES · NIST SP 800-115 · OWASP Testing Guide v4

---

## Confidentiality Statement

This document is the exclusive property of University of Plymouth Security (UPS). This document contains proprietary and confidential information. Duplication, redistribution, or use, in whole or in part, in any form, requires consent of both University of Plymouth Security and Sweeny Barbers.

Sweeny Barbers may share this document with auditors under non-disclosure agreements to demonstrate penetration test requirement compliance.

---

## Disclaimer

A penetration test is considered a snapshot in time. The findings and recommendations reflect the information gathered during the assessment and not any changes or modifications made outside of that period.

Time-limited engagements do not allow for a full evaluation of all security controls. The assessment prioritises the weakest security controls an attacker would exploit. Similar assessments should be conducted on an annual basis by internal or third-party assessors to ensure the continued success of the controls.

---

## Contact Information

| Name | Title | Contact |
|---|---|---|
| Nathan Clarke | Managing Director, Sweeny Barbers | n.clarke@plymouth.ac.uk |
| Toluwalase Oludipe | Student Lead Penetration Tester, UPS | student@postgrad.plymouth.ac.uk |

---

## Scope & Rules of Engagement

**In-Scope:** Internal network — `192.168.20.0/24`

**Excluded from scope (per client request):**
- Denial of Service (DoS)
- Phishing / Social Engineering

All other attack types were permitted by Sweeny Barbers.

**Client allowances:**
- Internal network access via port allowances across `192.168.20.0`

---

## Introduction

Cyber threats are increasingly becoming sophisticated and pervasive. As a result, organisations should identify and mitigate weaknesses before they are exploited. Penetration testing, also known as ethical hacking, is a controlled and authorised security assessment that simulates real-world attacks on systems, networks, or applications to uncover potential vulnerabilities (Scarfone & Mell, 2008). By imitating the Tactics, Techniques and Procedures (TTPs) of malicious actors, these tests provide critical insights into an organisation's security posture to facilitate decisions around risk management and remediation.

Sweeny Barbers is a regional hairdressing company with over 50 outlets. Following a decision by the Managing Director to digitise all operations, Sweeny Barbers engaged University of Plymouth Security (UPS) to conduct a penetration test to evaluate their security posture.

This report documents a penetration test aimed at evaluating the security of target systems within a controlled environment. The assessment follows recognised industry standards and methodologies, including guidance from the National Institute of Standards and Technology (NIST SP 800-115), ensuring a structured and ethical approach.

The objectives of this test were to enumerate active services, identify and analyse potential vulnerabilities, and where applicable, simulate exploitation to assess risk and provide mitigation strategies.

---

## Testing Methodology

The approach closely follows current industry standards, including the Penetration Testing Execution Standard (PTES), NIST SP 800-115, and the OWASP Testing Guide v4. All testing activities were conducted with explicit written authorisation and in accordance with legal and ethical guidelines. Testing was divided into six phases:

**1. Scanning and Enumeration** — Discovering live hosts, identifying active services, and mapping the network using Nmap ping sweeps and TCP SYN scans. OS fingerprinting was performed to identify operating systems running on hosts with the greatest number of exposed services.

**2. Vulnerability Analysis** — Matching gathered information against known vulnerabilities to assess risk. Targeted vulnerability scans were performed using Nmap NSE scripts, with manual CVE checks used to confirm findings. This phase bridges information gathering and exploitation, ensuring vulnerabilities are documented before any exploitation attempt.

**3. Gaining Access** — Executing the identified exploit using the Metasploit Framework. The EternalBlue exploit was launched against the identified target, resulting in remote code execution and shell access, confirming the presence of a high-impact vulnerability exploitable by real-world attackers.

**4. Maintaining Access** — Simulating attacker methods to sustain long-term access. Password hashes were extracted from the compromised system using post-exploitation tools and cracked using John the Ripper to reveal plaintext credentials, validating password reuse potential across other systems.

**5. Clearing Tracks** — Assessing whether an attacker could erase evidence of intrusion. Although logs were not deleted due to ethical constraints, commands and methods for log deletion were evaluated to highlight the importance of audit log protection and alerting systems.

**6. Reporting** — Documenting all findings with technical evidence, business impact context, and remediation recommendations aligned to current best practices.

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Parrot Security OS** | Attacker environment with pre-installed penetration testing tooling |
| **Nmap** | Host discovery, port scanning, OS fingerprinting, NSE vulnerability scripts |
| **Metasploit Framework** | Exploitation of MS17-010 via the EternalBlue module |
| **John the Ripper** | NTLM password hash cracking using the rockyou.txt wordlist |

---

## Evaluation

### Scanning and Enumeration

A list scan was performed to enumerate all available IP addresses on the `192.168.20.0/24` subnet using:

```bash
nmap -sL 192.168.20.0/24
```

The scan returned 256 available IP addresses ranging from `192.168.20.1` to `192.168.20.255`.

  <img width="452" height="175" alt="image" src="https://github.com/user-attachments/assets/458ca94c-21c8-4a30-93d3-1988f452035c" />

                   Fig 1: Scan for Available Hosts 

A SYN stealth scan was then performed across all hosts to identify those with open ports:

```bash
nmap -sS -Pn 192.168.20.0/24
```
  <img width="452" height="175" alt="image" src="https://github.com/user-attachments/assets/1d46738a-5d80-4bef-a379-4e885de32a48" />

            Fig 2: Scan for Open Ports on Available Hosts

The scan returned five hosts with open ports. All other hosts were unresponsive, filtered, or had closed ports.

   <img width="453" height="156" alt="image" src="https://github.com/user-attachments/assets/ed77f636-d251-4e49-a492-a1ad1136aa3e" />

                    Fig 3: 192.168.20.1

  <img width="458" height="246" alt="image" src="https://github.com/user-attachments/assets/8ed6a621-693d-40fb-bdbf-fb19732d8485" />

             Fig 4: 192.168.20.8 & 192.168.20.9 

<img width="452" height="328" alt="image" src="https://github.com/user-attachments/assets/93b82888-38b8-4c14-af32-f08a6f64921e" />

             Fig 5: 192.168.20.12 & 192.168.20.13 

#### Table of Results

| Target IP | Ports | Services |
|---|---|---|
| 192.168.20.1 | 53, 80, 443 | Domain, HTTP, HTTPS |
| 192.168.20.8 | 3389 | MS-WBT-Server (RDP) |
| 192.168.20.9 | 21, 80, 3389 | FTP, HTTP, RDP |
| 192.168.20.12 | 21, 23, 135, 139, 445, 49152–49158 | FTP, Telnet, MSRPC, NetBIOS-SSN, Microsoft-DS, Unknown |
| 192.168.20.13 | 445 | Microsoft-DS |

The host at `192.168.20.12` was identified as the priority target due to the greatest number of exposed services. A service and OS fingerprinting scan was then conducted:

```bash
nmap -sC -sV 192.168.20.12
```
  <img width="452" height="300" alt="image" src="https://github.com/user-attachments/assets/5eb7ec1a-2fd6-4573-b6dd-ecb261a1c6c5" />

          Fig 6: OS and Service Confirmation 
          
This revealed the target to be **Windows Server 2008 R2 Enterprise Service Pack 1 (x64)**, with NetBIOS name `SECAMWINSERVER2`.

---

### Vulnerability Analysis

A targeted vulnerability scan was performed using Nmap NSE scripts against the identified open ports:

```bash
nmap --script vuln -p 21,23,135,139,445 192.168.20.12
```

This identified **MS17-010**, a critical remote code execution vulnerability in Microsoft SMBv1 servers (CVE-2017-0143).

  <img width="466" height="312" alt="image" src="https://github.com/user-attachments/assets/998c9a2d-c5ab-4864-9c10-25e9174cc846" />

               Fig 7: Vulnerability Analysis 

Findings were cross-referenced against the Exploit Database to validate the vulnerability before proceeding to exploitation.

   <img width="452" height="118" alt="image" src="https://github.com/user-attachments/assets/8070b4d6-6ff4-4774-987f-564c8ab09edb" />

                  Fig 8: Exploit Confirmation 

---

### Gaining Access

The Metasploit Framework was launched and the MS17-010 EternalBlue module was located and configured:

   <img width="452" height="370" alt="image" src="https://github.com/user-attachments/assets/cd3d99f5-5ced-4de4-8e16-906bea193437" />

               Fig 9: Startup of Metasploit Framework

I then searched for the exploit using ms17-010 in the metasploit framework 

  <img width="501" height="140" alt="image" src="https://github.com/user-attachments/assets/9570493f-b194-4751-bcd4-5433188404e6" />

                      Fig 10: MS17-010 Search 

<img width="470" height="190" alt="image" src="https://github.com/user-attachments/assets/f271016b-f5a8-4959-b2e8-7cfa7507acd7" />

           Fig 11: Target Vulnerability Confirmation
           
The auxiliary scanner confirmed the target was vulnerable. The EternalBlue exploit module was then loaded and configured:


Then I proceeded to run a scan to verify the target was vulnerable using #setg RHOSTS 192.168.20.12 and then #run. The results indicated a vulnerability to MS17–010. 

  <img width="468" height="48" alt="image" src="https://github.com/user-attachments/assets/e4a0485e-bde6-4198-b386-62e6f913ba2d" />

      Fig 12: Subsequent Confirmation of Vulnerability in Metasploit 

I then proceeded to use the appropriate exploit module for MS17–010 using the command #use 0. Then set my payload with the #set windows/x64/meterpreter/bind_tcp command. Then I set my target host using the set RHOSTS 192.168.20.12 and #set RPORT 445 and set LPORT 4444 then #exploit.
 
  <img width="493" height="228" alt="image" src="https://github.com/user-attachments/assets/469f8221-8cb8-42a7-a3a8-1bbf71ce9e71" />

                     Fig 13: Exploit Activity 

Exploitation was successful, establishing a Meterpreter session on the target machine.

  <img width="452" height="207" alt="image" src="https://github.com/user-attachments/assets/39d60f0f-e85f-4c1f-a7f7-fdf5df7f898c" />

         Fig 14: Exploit Success 

**Note on payload selection:** The system recommended `windows/x64/meterpreter/reverse_tcp`, however `windows/x64/meterpreter/bind_tcp` was used due to initial connectivity constraints. With a reverse shell, the victim initiates the connection back to the attacker's listener. With a bind shell, the victim opens a listening port and the attacker connects to it — hence the connection direction `192.168.139.133:38939 -> 192.168.20.12:4444` seen in Fig 14.

---

### Maintaining Access

With the Meterpreter session active, NTLM password hashes were extracted from the compromised system:

  <img width="486" height="90" alt="image" src="https://github.com/user-attachments/assets/af775734-5b8f-4a6f-b20d-3b5d8bb8dc93" />

              Fig 15: Password Hash Extraction 

The extracted hashes were saved to `/home/student/Desktop/hashes.txt`.

   <img width="448" height="132" alt="image" src="https://github.com/user-attachments/assets/a5688ac8-675c-4be6-bf91-1c0b665ac39f" />

                    Fig 16: hashes.txt 

The `rockyou.txt` wordlist was downloaded and John the Ripper was used to crack the hashes:

  <img width="452" height="114" alt="image" src="https://github.com/user-attachments/assets/3454aeb2-da0a-401b-b069-22015101f5a2" />

                 Fig 17: Dictionary Download 

   <img width="452" height="112" alt="image" src="https://github.com/user-attachments/assets/6e6525e3-afd4-4fc0-863a-b3c958be055d" />

                    Fig 18: Password Crack 

| Command Part | Meaning |
|---|---|
| `john` | Runs John the Ripper, a password-cracking tool |
| `--format=NT` | Specifies the hash type as NTLM, used in Windows systems |
| `--wordlist=...` | Specifies the rockyou.txt wordlist for dictionary attack |
| `/home/student/Desktop/hashes.txt` | File path to the extracted hashes |

Cracked passwords were confirmed with:

```bash
john --show --format=NT /home/student/Desktop/hashes.txt
```

> 📸 *Fig 19: Password Confirmation — insert screenshot here*

**Result:** 4 of 5 password hashes were successfully cracked, demonstrating that weak, dictionary-guessable passwords were in use across multiple employee accounts.

---

## Key Insights

**Expanded Scanning Techniques:** Different Nmap scan types (`-sC`, `-sV`, NSE scripts) produced varying host-state outputs — some IPs initially appeared unresponsive until probed with a more aggressive SYN scan. This highlights the importance of using multiple scan types to avoid incomplete reconnaissance.

**Corroborating Vulnerability Results:** Although Nmap's `--script vuln` quickly flagged MS17-010, occasional false positives are possible. It is advisable to validate findings with complementary tools such as Nessus or OpenVAS before proceeding to exploitation.

**Exploit Delivery Tactics:** The vulnerability initially appeared not to be present in the system. The decision to use `bind_tcp` over `reverse_tcp` was driven by connectivity challenges encountered during the engagement, demonstrating the importance of adapting payload selection to real-world constraints.

**Deep Dive into EternalBlue Mechanics:** Hands-on insight was gained into how MS17-010 exploits SMBv1's kernel-level flaw to achieve remote code execution through port 445. Observations on EternalBlue's packet crafting and memory manipulation capabilities underscore the critical need for patching and SMBv1 deprecation.

---

## Understanding the EternalBlue Vulnerability

EternalBlue (MS17-010 / CVE-2017-0144) is a critical vulnerability in the Server Message Block (SMB) protocol used by Windows systems for sharing files, printers, and resources over a network. It was discovered in 2017 and is believed to have been developed for surveillance by the US National Security Agency (NSA) before being leaked publicly (Gupta, 2023).

The exploit targets a kernel-level flaw in the SMBv1 protocol. Kernel code is the core of an operating system and enforces the overall security model — any flaw at this level puts the entire system at risk (Scarfone et al., 2008). The attack operates through port 445, where the SMBv1 service listens. Once access is gained, the attacker can steal data, install ransomware, or pivot to other systems.

**Key Characteristics:**

| Field | Detail |
|---|---|
| Vulnerability ID | MS17-010 |
| CVE Identifier | CVE-2017-0144 |
| Affected Systems | Windows 7, Windows Server 2008, and earlier versions with SMBv1 enabled |
| CVSS v3.1 Score | 8.8 (Critical) |
| Impact | Unauthenticated remote code execution; full system compromise |

<img width="464" height="187" alt="image" src="https://github.com/user-attachments/assets/f0d455ea-5eb4-4b52-99f7-98f53b1f4706" />

           Fig 20: CVSS Rating of EternalBlue 

**Why This Matters to Sweeny Barbers:**

- **Rapid Network Propagation:** EternalBlue can spread across an entire network once a single machine is compromised. Liu (2021) examines how EternalBlue was used in ransomware such as WannaCry and NotPetya to achieve widespread infection — a single compromised host can bring down an entire organisation's network in minutes.
- **Data Breaches:** The exploit enables deep, persistent access to systems. Some malware variants are specifically designed to harvest credentials and extract sensitive data (Liu, 2021), leading to potential privacy violations and regulatory penalties.
- **Operational Downtime:** EternalBlue's ability to enable rapid lateral movement across networks causes widespread operational disruption, directly impacting revenue and customer service.
- **Reputational Damage:** A successful attack leads to erosion of customer trust and lasting damage to the organisation's reputation, with potential long-term financial consequences.

---

## Mitigation Recommendations

### 1. Patch Systems

The EternalBlue vulnerability was addressed by Microsoft in March 2017 with the release of patch **MS17-010**. All Windows systems, including legacy platforms such as Windows XP and Server 2003, should have this patch applied immediately.

A centralised patch management system should be implemented to automate and verify patch deployment. Mehri, Arlos and Casalicchio (2023) highlight that automated patching improves deployment speed and success rate while reducing human error and administrative overhead. Vulnerability scanning tools should be used to identify systems missing critical updates.

> 📸 *Fig 21: CVSS Rating — insert screenshot here*

### 2. Disable SMBv1

SMBv1 is an outdated protocol with no justification for continued use in modern environments. SMBv2 and SMBv3 provide equivalent functionality with significantly improved security.

SMBv1 can be disabled using Group Policy, PowerShell, or via the Windows Registry:

1. Press `Windows + R`, type `regedit`, and press Enter
2. Navigate to: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters`
3. Set or create a `DWORD` value named `SMB1` with data `0` (Disabled)

System administrators should audit any legacy applications that depend on SMBv1 and plan for their migration or replacement.

### 3. Network Segmentation

Implementing network segmentation limits the ability of malware to propagate across an environment. Critical assets — such as domain controllers, file servers, and production systems — should be isolated in separate network zones with tightly controlled access.

VLAN segmentation can separate user workstations from critical infrastructure, reducing the risk of a single point of failure (Naik et al., 2024). Port 445 should be restricted to only those systems that explicitly require it. Segmentation slows lateral movement and provides defenders with adequate time to respond before a full-scale compromise occurs.

### 4. Intrusion Detection and Prevention Systems (IDS/IPS)

IDS/IPS systems detect and respond to exploitation attempts such as EternalBlue in real time. Tools such as Snort and Suricata can monitor for unusual SMB traffic patterns, known exploit signatures, and suspicious behaviours. A layered deployment strategy is recommended, as no single system provides complete coverage (Riccardo et al., 2022).

### 5. Regular Backups

Backups should be performed frequently, stored in immutable formats, and tested regularly to ensure timely restoration. The **3-2-1 backup rule** (NIST, 2024) should be followed:

- Three copies of any important file: one primary and two backups
- Files stored on two different media types to protect against different hazard types
- One copy stored off-site

### 6. Least Privilege and Access Control

The penetration test demonstrated that over-provisioned administrative rights and shared accounts made it straightforward to crack passwords and escalate privileges. Adhering to the principle of least privilege significantly reduces the risk of unauthorised access and data breaches (Farman, 2024).

Recommended controls:
- Implement **Role-Based Access Control (RBAC)** to limit access to only what each user requires
- Deploy **Just-In-Time (JIT)** access models for administrative privileges
- Enforce **Multi-Factor Authentication (MFA)** across all accounts
- Disable the Guest account or enforce a strong, unique password

---

## Business Impact Analysis

In the event of a breach as demonstrated in this assessment, the following recovery metrics should be considered to ensure effective disaster recovery and business continuity planning:

**Recovery Time Objective (RTO):** The maximum acceptable period of downtime. A well-defined RTO guides the selection of appropriate recovery technologies and minimises financial impact during outages. RTO should align with the organisation's Maximum Tolerable Downtime (MTD) (NIST SP 800-34 Rev. 1, 2022).

**Recovery Point Objective (RPO):** The point in time to which data can be recovered following an outage, based on the most recent available backup (Swanson et al., 2010). A well-defined RPO ensures minimal data loss and maintains customer trust following an incident.

**Mean Time to Repair (MTTR):** A measure of how quickly systems can be restored after failure. Automated remediation workflows should be implemented to reduce MTTR and minimise operational impact.

**Mean Time Between Failures (MTBF):** An estimate of the average operational lifespan of a system component before failure, calculated as:

```
MTBF = Total Operational Time / Number of Failures
```

A higher MTBF indicates fewer disruptions, supporting smoother operations and better customer experience.

---

## Conclusion

This engagement demonstrated how an attacker can move from initial information gathering through to full post-exploitation, including credential harvesting and persistent access. Every phase was conducted within strictly defined legal and ethical boundaries — including explicit written authorisation, GDPR-aligned data handling, and controlled impact on production systems.

The use of Nmap for discovery, targeted NSE scripts for vulnerability validation, Metasploit's EternalBlue module for controlled exploitation, and John the Ripper for credential harvesting not only exposed the destructive potential of MS17-010, but validated the real-world feasibility of an SMBv1-based breach against an unpatched Windows Server environment.

EternalBlue can be thought of as a master key: once inserted, it unlocks every door in a networked environment, enabling attackers to move freely, exfiltrate data, and lock out legitimate users. To neutralise this threat, organisations must adopt a multi-layered defence:

- Enforce MS17-010 patching across all Windows systems
- Disable SMBv1 entirely
- Segment critical assets into isolated VLANs
- Deploy behavioural IDS/IPS tuned to detect anomalous SMB traffic
- Maintain immutable, air-gapped backups under the 3-2-1 regime
- Enforce least-privilege principles with RBAC and JIT access controls

Finally, embedding recovery metrics (RTO, RPO, MTTR, MTBF) into incident response playbooks ensures that, should the worst occur, operations can be restored swiftly and stakeholder trust preserved. In today's threat landscape, these steps are not optional best practices — they are mission-critical imperatives.

---

## References

1. Dorsey, B. (2014) *rockyou.txt – Popular Password Wordlist.* Available at: https://github.com/brannondorsey/naive-hashcat/tree/master
2. Farman, M. (2024) Implementing the Policy of Least Privilege: Enhancing Security through Segregation of Duties. *ResearchGate.*
3. GDPR (2018) Security of Processing — Article 32. Available at: https://gdpr-info.eu/art-32-gdpr/
4. Gupta, M.R. (2023) Eternal Blue Vulnerability. *IJRASET,* 11(6), pp. 1054–1060. doi: https://doi.org/10.22214/ijraset.2023.53795
5. Liu, Z. (2021) Working Mechanism of Eternalblue and its Application in Ransomworm. *arXiv.* doi: https://doi.org/10.48550/arxiv.2112.14773
6. Mehri, V., Arlos, P. and Casalicchio, E. (2023) Automated Patch Management: An Empirical Evaluation Study. Available at: https://bth.diva-portal.org/smash/get/diva2%3A1752783/FULLTEXT01.pdf
7. Naik, S. et al. (2024) A Review of the Role of Network Segmentation in Improving Cybersecurity and Preventing Data Breaches. *IJARSCT,* 4(3), pp. 27–35. doi: https://doi.org/10.48175/ijarsct-22806
8. NIST (2024) *Protecting Data from Ransomware and Other Data Loss Events.* U.S. Department of Commerce.
9. Nmap Project (2024) *Nmap — Network Mapper.* Available at: https://nmap.org/
10. Openwall Project (2024) *John the Ripper Password Cracker.* Available at: https://www.openwall.com/john/
11. Parrot Security (2024) *Parrot Security OS.* Available at: https://www.parrotsec.org/
12. Rapid7 (2024) *Metasploit Framework.* Available at: https://www.metasploit.com/
13. Riccardo, S. et al. (2022) *Analysis of Security Configuration for IDS/IPS.* Available at: https://webthesis.biblio.polito.it/secure/29003/1/tesi.pdf
14. Scarfone, K. et al. (2008) *NIST SP 800-115 — Technical Guide to Information Security Testing and Assessment.* Available at: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf
15. Swanson, M. et al. (2010) *NIST SP 800-34 Rev. 1 — Contingency Planning Guide for Federal Information Systems.* Available at: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf
16. The PTES Team (2022) *Penetration Testing Execution Standard.* Available at: https://www.pentest-standard.org/
17. UK Government (2018) *Data Protection Act 2018.* Available at: https://www.legislation.gov.uk/ukpga/2018/12/pdfs/ukpga_20180012_en.pdf

---

*This report was produced for academic purposes as part of the MSc Cybersecurity programme at the University of Plymouth. All testing was conducted within a controlled environment with explicit written authorisation.*
