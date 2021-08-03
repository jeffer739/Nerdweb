Hello, i made a writeup for the THM - Mustacchio room. Enjoy!!

![musttt](https://user-images.githubusercontent.com/64267672/127957099-fc60156d-19aa-4924-bfdc-e60049b637a2.png)


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




![mustetcxxe](https://user-images.githubusercontent.com/64267672/127956173-6c5cbdd8-c55c-4c1a-8ecc-46f07309c060.png)



and now the ssh key 

![mustsshxxe](https://user-images.githubusercontent.com/64267672/127956362-9ca4c792-359b-4ebc-a7f1-8ea24790973c.png)


now we can save the file locally and try to ssh but we find out the key password protected

For this we can use the ssh2john tool to convert the key to hash & then crack hash with john

![mustsshpas](https://user-images.githubusercontent.com/64267672/127955660-73c467d5-87ea-489b-bf39-7caaff17f184.png)


After changing the perms for the key Now we can ssh into the Barry machine successfully using the key & password and we can find our user.txt right there;


![mustuser](https://user-images.githubusercontent.com/64267672/127953264-4e9cfb9d-cf5e-4355-a890-3fddb82f4047.png)

Time to root!

Moving around & enumerating we find 2 users on the machine a an interesting binary on the Joe user;


Looking for suid bits set & we find some interesting stuff;

![mustsuid](https://user-images.githubusercontent.com/64267672/127953846-0251a17e-81ce-4594-87c6-5773108edf02.png)

Still enumerating, we find out we are able to execute the file so running strings on the file reveals to us that this binary runs the tail command without the absolute path. That makes me think of Path Variable attack.


![muststrings](https://user-images.githubusercontent.com/64267672/127954305-7ccb8d82-d325-43b6-963e-0c547c7ebee5.png)

Exploiting to root!

createa file in /tmp that executes /bin/bash and give it permission to run
export the PATH variable to point to the /tmp directory
Now run the file

![mustroot](https://user-images.githubusercontent.com/64267672/127954721-54d0b08c-60ca-4330-9612-028329a186f7.png)


### External refrences;
```
https://tryhackme.com/room/mustacchio
https://crackstation.net/
https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/
```
