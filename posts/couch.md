Hey guys welcome to my first blog post & hopefully I'm able to write & share more!! 
### Feedback's accepted :)

![couchprofile](https://user-images.githubusercontent.com/64267672/124205411-b2a2d280-dad0-11eb-8ebd-bed4a0d8668c.png)


Ok now we have the vm running we can kick off with an nmap scan....

```nmap -sV -A -p- -T4 -Pn 10.10.10.241 --max-retries=1 -oA nmap```


```
# Nmap 7.91 scan initiated Wed Jun 30 19:20:40 2021 as: nmap -sV -A -p- -T4 -Pn --max-retries=1 -oA nmap 10.10.200.74
Warning: 10.10.200.74 giving up on port because retransmission cap hit (1).
Nmap scan report for 10.10.200.74
Host is up (0.17s latency).
Not shown: 64093 closed ports, 1440 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:9d:39:09:34:30:4b:3d:a7:1e:df:eb:a3:b0:e5:aa (RSA)
|   256 a4:2e:ef:3a:84:5d:21:1b:b9:d4:26:13:a5:2d:df:19 (ECDSA)
|_  256 e1:6d:4d:fd:c8:00:8e:86:c2:13:2d:c7:ad:85:13:9c (ED25519)
5984/tcp open  http    CouchDB httpd 1.6.1 (Erlang OTP/18)
|_http-server-header: CouchDB/1.6.1 (Erlang OTP/18)
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 30 19:31:50 2021 -- 1 IP address (1 host up) scanned in 669.86 seconds
```

forging forward;

looking at the service at port 5984, we can get a list of the database;

we can access the couch web gui here,

![couchgui](https://user-images.githubusercontent.com/64267672/124209624-68721f00-dad9-11eb-8a83-f1ce7996a194.png)


![couchdbs](https://user-images.githubusercontent.com/64267672/124206842-ac622580-dad3-11eb-9fe0-b5b554d978bc.png)

ofc you're thinking the same as me cus a file "secret" from the list in the database looks sus so we take a look at it.

![credscouch](https://user-images.githubusercontent.com/64267672/124207785-ce5ca780-dad5-11eb-8ac9-4e648f880db8.png)


apparently we just found some creds by being curious, now what are the creds for? (take a look at the nmap scan results which shows port 22 being ssh open)
Did you just have the same thought as me again? Heccer!!

we can login ssh with those creds 

![couchssh](https://user-images.githubusercontent.com/64267672/124207992-2a273080-dad6-11eb-8617-70ed9bce1f2c.png)

Here we are!!

Let's get Root now.

digging into the kernel of the machine shows it's an old version 

![couchkernel](https://user-images.githubusercontent.com/64267672/124208350-e3860600-dad6-11eb-95da-ae9b2129db32.png)

Now exploiting Cve-2021-3493 to get root

![couchatenaexploit](https://user-images.githubusercontent.com/64267672/124208715-a8d09d80-dad7-11eb-8143-943969d39db8.png)

![couchroot](https://user-images.githubusercontent.com/64267672/124209016-5479ed80-dad8-11eb-83d1-262a00667674.png)

And we have content or root.txt at last & we can answer the rest of the task questions 🎏

![couchboomroot](https://user-images.githubusercontent.com/64267672/124209193-9c991000-dad8-11eb-8a19-e00b82638904.png)


### External refrences;
```
https://book.hacktricks.xyz/pentesting/5984-pentesting-couchdb
https://lzone.de/cheat-sheet/CouchDB
https://muzec0318.github.io/posts/overlayfs.html
https://github.com/briskets/CVE-2021-3493
```

