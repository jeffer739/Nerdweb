We start off with an nmap scan


![image](https://user-images.githubusercontent.com/64267672/145338498-e1adeaa8-6c01-4278-a321-4a77e2b5ba68.png)


Starting enumeration off from port 80 - http

We can see the home page;


![image](https://user-images.githubusercontent.com/64267672/145338666-84ce1e70-789c-4675-9dc6-655171a3891b.png)


We observe we are able to create an account & login

![image](https://user-images.githubusercontent.com/64267672/145338752-42b7863c-2102-438e-bf2e-be921a942e16.png)


And we are logged in as the user we just created;

![image](https://user-images.githubusercontent.com/64267672/145339853-7e823190-e88f-4a7f-93a3-54b310ac532a.png)


Looking at the profile page we see we are able to upload a profile picture BUT as an admin,

![image](https://user-images.githubusercontent.com/64267672/145340191-b861830e-266b-471b-97e4-5dc9845cbac9.png)



So we have to look for a way to be admin;

![image](https://user-images.githubusercontent.com/64267672/145340375-4ff86cc8-134a-47ea-a95e-17d67dcc6da9.png)



![image](https://user-images.githubusercontent.com/64267672/145340814-04eccdec-fd25-4915-9405-ff4ef4f89e4b.png)



![image](https://user-images.githubusercontent.com/64267672/145340945-28f1f77e-6afa-4b8b-ab07-c8aa00bbaa15.png)


We see we can change the admin password by replacing our email address with the admin address provided for us, ^^as shown;

now we can sign out & login with the admin account now


![image](https://user-images.githubusercontent.com/64267672/145343047-e930064d-61f7-49ed-aff5-cb2115cc5ff5.png)


And we have access to test the upload feature & we are def going to try to upload a malicous file to give us shell access

![image](https://user-images.githubusercontent.com/64267672/145343322-92945e85-e397-4977-8a3e-325abb0f896c.png)


Now we able to upload a php file & it got accepted, NO FILTER! (bad)

we look at our page source code & we get some hint possibly to where files like profile images are stored.

![image](https://user-images.githubusercontent.com/64267672/145343717-9cffb5ea-0bba-4cce-b4c6-fa3a5efcf5b1.png)


now we can access our malicious php reverse shell script while we set our listener up waiting for a connection 

![image](https://user-images.githubusercontent.com/64267672/145344558-43a8ec79-e3df-4584-9358-86f58b301ecd.png)


We got a shell as well as prove as user.txt,


We enumerate again to escalate priviledge;


Met some rabbit holes while enumerating this one, but looking services running locally & something interesting here;

![image](https://user-images.githubusercontent.com/64267672/145364042-49e21e65-2478-4091-b8fa-c2dd5491dbd5.png)


SQL running on port 33060 & MongDB running on port 27017. 
Time to open MongoDB in terminal.

![image](https://user-images.githubusercontent.com/64267672/145372357-9511d430-94c1-4987-827b-ed6a2bbe79f5.png)


And we find some credentials, so i think we can test now on ssh;



![image](https://user-images.githubusercontent.com/64267672/145367825-58746e01-4986-4b1f-90ed-a575bf7e0a9b.png)

now we have access to test for sudo privileges & doing that we find we that webdeveloper can run  /usr/bin/sky_backup_utility as sudo also without password





![image](https://user-images.githubusercontent.com/64267672/145377629-54f176ec-6ecd-4d54-b2d2-60f2e4826ce3.png)




/etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf contains AdminIdentities=unix-group:sudo;unix-group:admin (Default in Ubuntu)

pkexec allows an authorized user to execute commands as another user & as webdeveloper is a member of the sudo group then we can use psexec


![image](https://user-images.githubusercontent.com/64267672/145372687-5ac46fc6-a924-441f-9390-ebc896cd857f.png)



And we are root! 😉






Figured we can root this machine another way, so let's see how.






So back to webdeveloper user shell;


![image](https://user-images.githubusercontent.com/64267672/145373287-b15c8a7d-f9af-46e2-b0a1-b4df2d43c299.png)



sudo -l shows /usr/bin/sky_backup_utility has LD_PRELOAD explicitly defined in the sudoers fileed


![image](https://user-images.githubusercontent.com/64267672/145373432-ab5c024b-624e-4877-97df-8c7904a99ea3.png)


now we can write our C code & compile with gcc -fPIC -shared -o shell.so shell.c -nostartfiles


![image](https://user-images.githubusercontent.com/64267672/145373761-67c78a93-7d2d-44e5-9923-cdb0130e1a44.png)


Now we execute it with sudo LD_PRELOAD=/home/webdeveloper/shell.so sky_backup_utility and we are root! 

![image](https://user-images.githubusercontent.com/64267672/145375217-a165e1c2-b1c7-4226-b006-66999042e412.png)

