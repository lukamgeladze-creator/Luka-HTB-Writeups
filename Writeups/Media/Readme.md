# 📺 HTB Media
```
Machine Information:
Media is a Medium difficulty machine that features an Apache XAMPP stack on Windows hosting a custom PHP web application. The web application allows the upload of a Windows Media Player compatible file that can be leveraged to leak the NTLMv2 hash of the user account that opens it. This hash can be cracked to obtain user credentials that can be used to authenticate to the target via SSH. Upon gaining initial access the source code of the application can be analyzed to determine the generate storage path of uploaded files on the web application which can lead to an NTFS Junction (directory symbolic link) attack to upload a malicious PHP web shell for RCE. Once a shell under the context of the web server's service account, players can abuse the SeTcbPrivilege - Act as part of the operating system, a Windows privilege that lets code impersonate any user and achieve administrative privileges. Alternative methods for privilege escalation involve regaining the SeImpersonate privilege to elevate to NT Authority\SYSTEM.
```
# 📝 Summary
Media is a medium difficulty machine that focuses on knowing xampp, junction and how to escalate privileges with SeTcbPrivilege.
- We are starting with finding that in the webserver there is an upload button which only allows .asx files to be uploaded.
- After uploading a malicious .asx file they return a ntlm hash for the user enox.
- We get the first flag in the enox desktop and then we need to discover where exactly are the files uploading so that we are able to use junction attack
- After locating junction point and figuring out how the hashes are stored in the MD5 we are able to upload the malicious php web shell which would give us authority of nt authority\local service
- After that we have SeTcbPrivilege as a way to escalate privileges, so we transfer FullPowers.exe, GodPotato-NET4.exe and nc.exe and we get a reverse shell back granting us the administrator rights.

# 🔎 Recon
First i always do the aggressive scan because it is more effective way to discover most of the information(If the ports are not hidden or changed)
```
sudo nmap -sV -A -sV 10.129.234.67 -oA media                      
[sudo] password for kali: 
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-22 03:43 -0500
Nmap scan report for 10.129.234.67
Host is up (0.16s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
80/tcp   open  http          Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.1.17)
|_http-title: ProMotion Studio
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.1.17
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-11-22T08:43:54+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=MEDIA
| Not valid before: 2026-11-21T08:33:37
|_Not valid after:  2027-05-23T08:33:37
| rdp-ntlm-info: 
|   Target_Name: MEDIA
|   NetBIOS_Domain_Name: MEDIA
|   NetBIOS_Computer_Name: MEDIA
|   DNS_Domain_Name: MEDIA
|   DNS_Computer_Name: MEDIA
|   Product_Version: 10.0.20348
|_  System_Time: 2026-11-22T08:43:49+00:00
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```



