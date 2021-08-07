Hello, I recntly completed the THM - CMSpit room on TryhHackMe in which i exploited a recent cves to get both foothold & root here & now making a walkthough, Enjoy!

CVE-2021-22204 - FootHold
CVE-2021-22204 - Root

![cmspitbanner](https://user-images.githubusercontent.com/64267672/128586316-fbf0b26e-91f4-4864-aa18-6783ed4beaa6.png)

Here we start off with an nmap scan & we find 2 ports opened;
![cockpitnmap](https://user-images.githubusercontent.com/64267672/128586669-b4241e78-39b7-424d-8d73-f310b9d31da6.png)


Now looking at the web page on port 80 we find a login page of a cms;
![cockpitlogin](https://user-images.githubusercontent.com/64267672/128586461-71ce9f7c-2bb3-4d5f-bfb6-64aead14c1a8.png)

Now after some research(google) we find out that this cms has multiple vulnerabilities,
We find a good blog explaiing how the vulnerabilities work & how to exploit it(see refrence)


Now let's get started with finding valid users.
![cockpitusers](https://user-images.githubusercontent.com/64267672/128586559-4ef12f97-c82b-4fa5-8c3b-0f489e3d0eb7.png)

Now compromising the users;
Firstly we can generate a resetpassword token for each users;
![cockpitresettokens](https://user-images.githubusercontent.com/64267672/128586707-3213b09d-6c8f-4f5f-99a1-127e46c38215.png)

We can query each token to return to us more User information(username, email, password hash, API key, password reset token)
![cockpitskidyemail](https://user-images.githubusercontent.com/64267672/128586852-2a111af2-c349-4ce5-a869-813261fc334e.png)

Now we have the token we can change user password with token;
![cockpitpasswordrest](https://user-images.githubusercontent.com/64267672/128586935-6dabf781-0443-4396-8ee2-317c4726e6ec.png)

Now login in with changed password (my case "Newpassword123");
![cockpitadminui](https://user-images.githubusercontent.com/64267672/128586997-c541f58f-9e63-4df5-9610-6d327e95f2c2.png)

we can find the web flag by browsing to /finder directory of the web page;
![cockpitwebflag](https://user-images.githubusercontent.com/64267672/128587021-96a26bf5-e463-4308-aadf-bc7c92a38b49.png)

After studying the web app we discover we are able to upload new files,
So we are going to upload a webshell here & eventually a reverse shell waiting in our netcat listener;
![cockpitshellscript](https://user-images.githubusercontent.com/64267672/128587131-14e92f9d-1b6d-4fc4-8e30-55758fa052e4.png)

We can confirm our webshell by runing a simple linux "id" command;
![cockpitwebshell](https://user-images.githubusercontent.com/64267672/128587160-13b198e3-51c8-4444-aafe-bb6b10d71d09.png)

Now we can upload our reverse shell to get our RCE waiting on netcat listener here;
![cockpitnetshell](https://user-images.githubusercontent.com/64267672/128587197-04afa80b-25cf-4002-a32c-fb236ca945cf.png)

Now we can stablize the shell & enumerate this www-data user, 
After running linpeas we find an interesting mongoDB laying around, Yes we'll enumumerate that & we find some creds.
![cockpitmongopass](https://user-images.githubusercontent.com/64267672/128587269-804f3e7a-c869-4e6b-91da-b615b905c748.png)

we can ssh into the user machine we've just discovered with password;
![cockpitssh](https://user-images.githubusercontent.com/64267672/128587297-ede60ae4-921a-4a0e-9ea6-7735b91daac6.png)

We have our user.txt here & we can now escalate our privileges to root.
Running the sudo -l command we can see that stux user could execute the exiftool as the root without requiring root password (CVE-2021-22204)
Visiting gtfobins and searching on the binary name we see this tool can be used to read and write files. We change it to read from a file and output it to a file in a directory we can read from.

so we run;
sudo exiftool -filename=/home/stux/root.txt /root/root.txt

And dang! we have a root.txt file from root user.
![cockpitroottxt](https://user-images.githubusercontent.com/64267672/128587695-98db7049-0e08-4839-ae31-de607efd221e.png)

Exploiting CVE-2021-22204 to get root?


https://github.com/convisoappsec/CVE-2021-22204-exiftool


https://github.com/AssassinUKG/CVE-2021-22204

### REFRENCES

https://tryhackme.com/room/cmspit


https://swarm.ptsecurity.com/rce-cockpit-cms/


https://gtfobins.github.io/gtfobins/exiftool/


https://github.com/convisoappsec/CVE-2021-22204-exiftool


https://github.com/AssassinUKG/CVE-2021-22204
