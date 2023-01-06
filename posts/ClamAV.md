Starting with nmap scan here


![image](https://user-images.githubusercontent.com/64267672/210963067-635798b8-15bd-4b1c-a527-da9f5379517e.png)


We have so many open ports to enumerate but with not much interesting, i have to search for the name of the machine on searchsploit cus it look familiar


![image](https://user-images.githubusercontent.com/64267672/210964023-504dab50-d04e-4ece-b7e5-a06f83edd8b3.png)

The last exploit(4761.pl) is what we can use to get foothold 

![image](https://user-images.githubusercontent.com/64267672/210964188-87fbda49-7a9b-4591-9107-71e3bf4aad78.png)

removing the comments & adding perl shebang #!/usr/bin/perl

reading the exploit & we can see it executes sh as root & opens the port 31337 so we also have to connect through that port on our machine

now we can run the exploit 

![image](https://user-images.githubusercontent.com/64267672/210964731-805ba5e6-3300-4bf1-b67b-033d446789e0.png)


Now connect with netcat through the port 31337 

![image](https://user-images.githubusercontent.com/64267672/210965493-02862ffb-e674-4cf7-8ff8-fb213876d54d.png)


And we are root automatically, no need to upgarde privs
