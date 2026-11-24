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






