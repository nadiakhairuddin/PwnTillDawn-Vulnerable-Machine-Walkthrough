# PwnTillDawn-Vulnerable-Machine-Walkthrough-.11

**Target IP:** 10.250.250.11  
**Scope:** Reconnaissance, Enumeration, Exploitation, Privilege Escalation

## Reconnaissance

The first step is to establish a connection to the PwnTillDawn lab environment using OpenVPN.

```bash
sudo openvpn PwnTillDawn.ovpn
```
Once the VPN connection is successfully established, the target machine (10.250.250.11) becomes accessible for further scanning and enumeration.

**OpenVPN connection established successfully:**

<img width="733" height="628" alt="Screenshot 2026-04-01 213801" src="https://github.com/user-attachments/assets/a975c6a6-847c-4973-88ee-d342d32d776c" />
<img width="731" height="514" alt="Screenshot 2026-04-01 213935" src="https://github.com/user-attachments/assets/9759cb7d-12dc-41fe-87be-1a105e1c0118" />

A ping sweep is performed using Nmap to discover active hosts:

```bash
nmap -sn 10.250.250.0/24
```
The target IP (10.250.250.11) is identified from the scan results.

<img width="621" height="645" alt="image" src="https://github.com/user-attachments/assets/e2eb79be-213d-412c-8882-1166ce46c8b7" />

## Scanning / Enumeration

A detailed Nmap scan is performed to identify open ports, running services, and service versions on the target machine.

```bash
nmap -sC -sV -Pn -vv 10.150.150.11
```
The Nmap scan was performed with the following options:

- `-sC` runs default scripts to gather additional information  
- `-sV` detects service versions  
- `-Pn` skips host discovery (assumes the host is up)  
- `-vv` enables verbose output for more detailed results  

These options help in performing a more in-depth enumeration of the target system.

The scan results provide important information about the target, including open ports, running services, and their versions. This data is essential for identifying potential vulnerabilities for further exploitation.

**Nmap scan results:**

<img width="620" height="647" alt="image" src="https://github.com/user-attachments/assets/bdce98f0-8df0-4a10-95ae-d1107da3c950" />

## Scanning → Directory Enumeration (Gobuster)

Directory enumeration is performed using Gobuster to discover hidden directories and potential entry points on the web server.

```bash
gobuster dir -u http://10.150.150.11 -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
```
This scan helps identify hidden paths that are not directly visible through the web interface, such as admin panels or upload directories.

From the results, several interesting directories were discovered that may be potential attack vectors, including `/upload` and `/admin.`

**Gobuster scan results:**

<img width="754" height="691" alt="image" src="https://github.com/user-attachments/assets/3282dd6d-c07a-4cd4-9b1b-845e617b7587" />

The discovered directories are further analyzed to identify possible vulnerabilities, especially in areas such as file upload functionality and administrative access points.

## Accessing Admin Panel & User Creation

The `/admin` directory discovered during enumeration is accessed through the web browser.

Upon visiting the admin page, an administrative interface is found which includes a feature to add new users.

**Admin panel accessed:**

<img width="790" height="449" alt="Screenshot 2026-04-01 222144" src="https://github.com/user-attachments/assets/2e590875-f55e-4dca-bb95-719cc308136c" />

A new user is then created through the "addedituser" which is "Add User" functionality. During this process, the user is assigned administrative privileges.

**Adding new user as admin:**

<img width="869" height="668" alt="image" src="https://github.com/user-attachments/assets/75988052-ff6c-450c-9f48-fe14f1cbd9f0" />

After successfully adding the user, confirmation is displayed indicating that the account has been created with admin access.

<img width="985" height="830" alt="image" src="https://github.com/user-attachments/assets/6657b0b0-a7ab-4ac6-9404-5ce8c16aeb9f" />

**User creation successful:**

<img width="979" height="833" alt="image" src="https://github.com/user-attachments/assets/1903e323-e698-4677-9ad5-d62e524a6b80" />

## File Upload (Shell Upload)

After gaining access to the admin panel, the file upload feature is tested for potential exploitation. A PHP reverse shell file (`shell.php`) is uploaded to the server through the upload functionality.

`shell.php`

The file is successfully uploaded, indicating that there is no proper restriction on file type validation.

**Shell file uploaded successfully:**

<img width="977" height="832" alt="Screenshot 2026-04-01 223843" src="https://github.com/user-attachments/assets/70092b7f-b5c7-4855-aaf4-aa3aefccdded" />

## Uploaded File Location Discovery

After the upload is completed, the upload directory is enumerated to locate where the file is stored.

The main upload directory is accessed:

```bash
http://10.150.150.11/upload/
```

From here, multiple subdirectories are checked to identify where the uploaded file is placed. It is discovered that the file is located in directory **/11/**.

**Upload directory listing:**

<img width="952" height="663" alt="image" src="https://github.com/user-attachments/assets/59c18a18-fbdf-451c-b2e2-363e8083acd0" />

The identified path is then confirmed:

```bash
http://10.150.150.11/upload/11/
```

**Shell location confirmed:**

<img width="946" height="664" alt="image" src="https://github.com/user-attachments/assets/19282275-6c6a-40bd-81f5-75a5727fcd2d" />

## Command Execution via Web Shell

The uploaded shell is accessed to verify command execution capability using the `whoami` command

```bash
http://10.150.150.11/upload/11/shell.php?cmd=whoami
```
The response confirms that command execution is working on the target system.

**Command execution result:**

<img width="830" height="510" alt="image" src="https://github.com/user-attachments/assets/2fa21f61-cfcc-4a3b-964a-1531c990c1ae" />

The output shows that the current user is `nt authority\system`, indicating that the shell is running with high-level privileges.

## Post Exploitation Reconnaissance

After gaining command execution on the target system, further information is gathered using the `systeminfo` command to understand the system environment.

```bash
http://10.150.150.11/upload/11/shell.php?cmd=systeminfo
```
The output provides detailed information about the target machine, including OS version, architecture, and network configuration.

**System Information Retrieved:**

- **Host Name:** PWNDRIVE
- **OS:** Microsoft Windows Server 2008 R2 Enterprise
- **OS Version:** 6.1.7601 Service Pack 1 (Build 7601)
- **System Type:** x64-based PC
- **Manufacturer:** VMware, Inc. (Virtual Machine)
- **Processor:** Intel64 Family 6 (6 cores, ~2194 MHz)
- **Total Physical Memory:** 8 GB
- **Available Memory:** ~7 GB
- **Time Zone:** (UTC-08:00) Pacific Time
- **Hotfixes Installed:** KB2999226, KB976902
- **Network IP Address:** 10.150.150.11

**System Information Output:**

<img width="953" height="665" alt="image" src="https://github.com/user-attachments/assets/efc381e6-a8f6-4baa-ae75-28e43adc58a1" />

This information is useful for identifying potential vulnerabilities based on the operating system version and missing patches, which can be used for further privilege escalation.

Further enumeration is performed on the target system to explore the file structure and identify potential user accounts.

```bash
http://10.150.150.11/upload/11/shell.php?cmd=dir C:\
```

The command is used to list directories in the C drive, specifically focusing on user-related folders.

**Directory Listing of C:\Users:**

<img width="956" height="665" alt="image" src="https://github.com/user-attachments/assets/6018e50d-86b5-4cab-9d6b-2e1342aa9128" />

### Key Findings:

- Multiple user accounts exist on the system:
  - Administrator  
  - Jboden  
  - Public  
  - tony  

- Microsoft SQL Server (SQLEXPRESS) is installed, indicating a database service running on the system.

- The system is a multi-user Windows Server environment, making it a suitable target for privilege escalation.

- IIS components are present, suggesting web services are also running on the machine.

## Sensitive File Discovery

Further enumeration is performed on the Administrator’s desktop directory to identify any sensitive files or flags.

```bash
http://10.150.150.11/upload/11/shell.php?cmd=dir%20C:\Users\Administrator\Desktop
```

The output reveals files located on the Administrator’s desktop.

**Administrator Desktop Directory Listing:**

<img width="949" height="666" alt="image" src="https://github.com/user-attachments/assets/e2720c4c-f677-4181-b80f-95fa0b46804d" />

Key Findings:
- `FLAG1.txt` is present on the Administrator’s desktop
- Shortcut file `Xlight FTP Server.lnk` is also found
- Presence of FLAG file indicates successful path toward completing the challenge

This confirms that sensitive information is stored within the Administrator user directory and can be accessed through command execution.

## Flag Capture

After identifying the presence of `FLAG1.txt` on the Administrator’s desktop, the file content is retrieved using the following command:

```bash
http://10.150.150.11/upload/11/shell.php?cmd=type C:\Users\Administrator\Desktop\FLAG1.txt
```
The output reveals the hidden flag stored inside the file.

**Flag retrieved:**

<img width="951" height="666" alt="image" src="https://github.com/user-attachments/assets/0ffba4d4-6cef-4552-998a-d38b66797dbb" />

The flag obtained is:

`PwnTillDawnAcademyIsAwesome!!!`

This confirms successful exploitation of the target system and completion of the challenge.

## 💡 Hacking Methodology (6 Stages)

In general, a typical penetration testing process consists of 6 stages:

- Reconnaissance ✔  
- Scanning ✔  
- Gaining Access ✔  
- Privilege Escalation ✔  
- Maintaining Access (optional)  
- Clearing Tracks (optional / reporting stage)  

👉 However, for CTF or lab environments:

❗ Stages 5 and 6 are usually not required, as the main objective is to gain access and capture the flag.

## Conclusion

This walkthrough demonstrates the successful exploitation of the target machine in the PwnTillDawn lab environment (10.250.250.11).

Starting from reconnaissance and service enumeration, critical information was gathered using tools such as Nmap and Gobuster, which revealed open services, hidden directories, and potential attack surfaces.

By exploiting a vulnerable file upload feature in the admin panel, a web shell was uploaded successfully, allowing remote command execution on the target system. This led to full system access with high-level privileges (`nt authority\system`).

Further post-exploitation enumeration revealed important system information, user accounts, and sensitive files located on the Administrator’s desktop. Ultimately, the flag was retrieved successfully from the system.

## What Was Achieved

- Successful VPN connection to the lab environment  
- Identification of target IP via network scanning  
- Discovery of hidden directories using Gobuster  
- Exploitation of admin panel user creation functionality  
- Successful upload and execution of a PHP web shell  
- Remote command execution on the target system  
- System-level access (`nt authority\system`) achieved  
- Retrieval of sensitive information and system flag  

## Final Result

✔ Target compromised successfully  
✔ Flag captured: `PwnTillDawnAcademyIsAwesome!!!`  

This confirms full completion of the PwnTillDawn vulnerable machine challenge.
