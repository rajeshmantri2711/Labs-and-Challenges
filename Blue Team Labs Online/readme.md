# Blue Team Labs – Practical Investigations

This section documents hands-on Blue Team investigations completed through practical lab-based challenges.
Each topic includes the investigation context, key learning outcomes.

---

## Employee of the Year

**Activity:**  
Analyzed a disk image to investigate deleted and hidden artifacts related to the incident. Used multiple forensic utilities to extract files, recover metadata, and identify traces of user activity.

**Learning:**  
- Disk image analysis fundamentals  
- File carving using `foremost`  
- PDF artifact analysis with `pdf-parser`  
- Recovering deleted file names using `strings`  
- Understanding filesystem layout and artifacts  

**Proof:**  
- [Employee_of_the_year](Employee_of_the_year.png)

---

## Follina

**Activity:**  
Analyzed a malicious Microsoft Word document (`sample.doc`) associated with the Follina (CVE-2022-30190) vulnerability. Since the original malware infrastructure was no longer active, dynamic analysis platforms such as Any.Run were not viable. To complete the investigation, relied on open-source intelligence and publicly available threat research.

**Learning:**  
- Understanding the Follina (CVE-2022-30190) vulnerability  
- Analyzing malicious Office documents  
- Using OSINT and threat intelligence reports for malware analysis  
- Interpreting real-world incident response write-ups  

**Proof:**  
- [Follina](Follina.png)

---

## Log Analysis – Privilege Escalation

**Activity:**  
Performed log analysis to identify privilege escalation attempts following initial compromise. Traced attacker activity from low-privileged access to elevated permissions.

**Learning:**  
- Privilege escalation indicators  
- Timeline reconstruction  
- Authentication and system log analysis  

**Proof:**  
- [Log_Analysis_Privilege_Escalation](Log_Analysis_Privilege_Escalation.png)

---

## Meta

**Activity:**  
Investigated metadata artifacts within files and logs to uncover attacker activity and potential data exposure.

**Learning:**  
- Metadata analysis techniques  
- Artifact based investigation  

**Proof:**  
- [Meta](Meta.png)

---

## Network Analysis – Ransomware

**Activity:**  
Analyzed network traffic associated with a ransomware incident. Identified C2 communication and attacker movement across the network.

**Learning:**  
- Ransomware network indicators  
- Malicious traffic patterns  
- Network-based detection  

**Proof:**  
- [Network_Analysis_Ransomware](Network_Analysis_Ransomware.png)

---

## Network Analysis – Web Shell

**Activity:**  
Investigated suspicious web server traffic linked to a deployed web shell. Analyzed HTTP requests to identify attacker interaction and command execution.

**Learning:**  
- Web shell behavior  
- HTTP traffic analysis  
- Web server log investigation  

**Proof:**  
- [Network_analysis_web_shell](Network_analysis_web_shell.png)

---

## Phishing Analysis

**Activity:**  
Analyzed a phishing email to determine legitimacy by reviewing headers, payloads, and infrastructure used by the attacker.

**Learning:**  
- Email header analysis  
- Phishing indicators  
- Social engineering techniques  

**Proof:**  
- [Phishing_Analysis](Phishing_Analysis.png)

---

## Phishing Analysis – Advanced

**Activity:**  
Performed advanced phishing investigation involving multiple payloads, redirects, and attacker infrastructure.

**Learning:**  
- Advanced phishing techniques  
- URL and attachment analysis  
- Infrastructure tracking  

**Proof:**  
- [Phishing_Analysis_2](Phishing_Analysis_2.png)

---

## PowerShell Analysis – Keylogger

**Activity:**  
Investigated malicious PowerShell scripts used to deploy a keylogger. Analyzed script execution and persistence techniques.

**Learning:**  
- Malicious PowerShell patterns  
- Script obfuscation  
- Endpoint detection opportunities  

**Proof:**  
- [PowerShell_Analysis_Keylogger](PowerShell_Analysis_Keylogger.png)

---

## Reverse Engineering – A Classic Injection

**Activity:**  
Analyzed a malware sample using classic process injection techniques to understand execution flow and evasion behavior.

**Learning:**  
- Process injection concepts  
- Malware execution techniques  
- Memory-based attack indicators  

**Proof:**  
- [Reverse_Engineering_A_classic_Injection](Reverse_Engineering_A_classic_Injection.png)

---

## Secrets

**Activity:**  
Investigated exposed secrets in files and configurations to identify credential leakage and security misconfigurations.

**Learning:**  
- Secret discovery techniques  
- Credential exposure risks  
- Secure configuration practices  

**Proof:**  
- [Secrets](Secrets.png)

---

## The Report

**Activity:**  
Compiled a full incident response report documenting findings, timelines, impact assessment, and remediation recommendations.

**Learning:**  
- Incident documentation  
- Reporting for stakeholders  
- Clear technical communication  

**Proof:**  
- [The_Report](The_Report.png)

