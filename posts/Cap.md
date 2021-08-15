Here's my first HackTheBox writeup "Cap".

Starting off with an nmap scan;

![cap nmap](https://user-images.githubusercontent.com/64267672/129467525-4faf8e34-eafb-4ad1-b12e-78d9c9c71dca.png)

I started by looking at the ftp port and noticed it doesn't allow annonymous login so i need to find the credentials for ftp login.
I visited the web page on port 80 & found an interesting webpage with so much info 
I also found so many files in /data Directory & after poking around i found an interesting file called 0.

So i downloaded the pcap file & analyzed with wireshark 

![capsharkcred](https://user-images.githubusercontent.com/64267672/129468090-aec3e9a2-3909-4604-a190-f6ffbf04e16d.png)


Here we find some creds.

i tried to login ftp with it & it got me acess inside.


![capftp](https://user-images.githubusercontent.com/64267672/129468135-7b5ded77-956b-4e35-8099-072a4b5ab478.png)

There i got access to other files including user.txt.

i discovered i could use the same creds to login with ssh


![capsshuser](https://user-images.githubusercontent.com/64267672/129468141-0779b123-df5f-4d50-b157-02fbef258e73.png)


Now running Linpeas on target machine & i found /usr/bin/python3.8 with an intersting capability bit set

![cap sudocapability](https://user-images.githubusercontent.com/64267672/129468294-ea3da0f8-7e7f-4ec1-b1a6-4cca50c5b670.png)


now looking at GTFObins i can run the command " python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")' " to get root!

![caproot](https://user-images.githubusercontent.com/64267672/129468299-cd4c0c8b-8a74-4518-a4bf-c5ccc0e37bf3.png)


### REFRENCES


https://app.hackthebox.eu/machines/351

https://gtfobins.github.io/gtfobins/python/
