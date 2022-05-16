The Examiner

![image](https://user-images.githubusercontent.com/64267672/168491434-1026fb77-1930-41f1-87d2-ceb599eeaff0.png)

This is a private TryHackMe Machine 

![image](https://user-images.githubusercontent.com/64267672/168491467-74b9ae4a-9948-4d35-8bc2-78c8d435aa74.png)

We start with an nmap scan to detect open ports 

![image](https://user-images.githubusercontent.com/64267672/168492373-30201966-4999-422e-aef6-4dd2aee24774.png)

We have just 1 open port 5000

Port 5000 runs http on Nodejs

Visiting the webpage on port 5000, we have somewhat a port scanner

![image](https://user-images.githubusercontent.com/64267672/168492468-0cc0e271-fb7c-4aa1-a7ab-2fff8265ea57.png)

When we check the page source, we can find an interesting javascript code that runs backend 

![image](https://user-images.githubusercontent.com/64267672/168492518-19025bdc-28d5-4c21-949f-162e5947c10c.png)

This code makes a post request to /scan & checks that we have to give a host to scan for open port & also a command to execute

I try to test for some command injection (Thanks HTB Routerspace)

![image](https://user-images.githubusercontent.com/64267672/168494615-7e6b0206-ac5c-4fe6-91fa-c9a9edb3512d.png)

"Port is closed" weird

I tried to bypass filter & also combine Ip, port & command to execute & i get "Enter the IP:PORT you want to scan." weird

![image](https://user-images.githubusercontent.com/64267672/168494968-7b902ca9-ce34-4a1f-b8a6-d0320c31290a.png)


Some filtering were being done; give it an ip addreess, a port & a command to execute & it gets filterd 

![image](https://user-images.githubusercontent.com/64267672/168494296-025953a6-0fe3-4642-90bd-e4a8e41659b4.png)

Weird

Now could make a python script to try all the three letter combinations

![image](https://user-images.githubusercontent.com/64267672/168495043-0f344625-95c7-4600-859f-b26417830ac4.png)


Now we find another weird & interesting result 


![image](https://user-images.githubusercontent.com/64267672/168495142-bd905058-0dd4-49e1-b6fb-960462fdf913.png)


Give it a 3 character string, a port & command to execute & it gets filtered again 

![image](https://user-images.githubusercontent.com/64267672/168494381-aa3605db-a404-4958-89c7-ec3dc2a22452.png)


But works with any two chareacter random string, a port & a command to execute & it works fine 


![image](https://user-images.githubusercontent.com/64267672/168494116-3e89a624-7a1d-4f34-a866-b66faa5d3b1f.png)

now we have to look for a way to get remote code execution & a shell on system 

We use a bash reverse shell & connect back through netcat & we got shell & we also can stablise shell now 

![image](https://user-images.githubusercontent.com/64267672/168495530-a4564363-f1e8-416b-a34c-fe897cbc6875.png)

Looking for flag now 

switching to the home directory & can see 2 different users folder but we need sudo permission to switch to the sshuser user

![image](https://user-images.githubusercontent.com/64267672/168518042-9fc37f4b-d154-489b-aca0-de4769e4ad9f.png)


so time to priv esc

running sudo -l shows us that Rudefish user can run /usr/bin/env as sudo and with help of gtfobins we can become root by abusing this privilege

![image](https://user-images.githubusercontent.com/64267672/168496253-3444c513-1d25-45dc-8053-0a2e2ad41b78.png)

And we are root!

![image](https://user-images.githubusercontent.com/64267672/168496288-c92f04d8-1bfa-4af0-9cbc-16809e3df9be.png)

I switched back to the sshuser directory & got nothing there so switched back to Rudefish directory & listed all files then i saw an interesting .ssh directory with an id_rsa & id_rsa.pub file


Now i am thinking of connecting back via ssh but my nmap scan doen't detect any ssh open port at first so i think i can scan the internal network for open ports now i have access as Rudefish, sshuser & root. Using the "netstat -tulpn" command as root user on Rudefish directory i can see an open port 2220 running sshd

![image](https://user-images.githubusercontent.com/64267672/168519600-5344347d-b141-4cb5-b91e-13d9c7ee798c.png)

listing all files on root on /root directory & we have .bash_history & .ssh interesting 

We can read the files in .bash_history but we just found out the flag got removed amongst some other information giving us the idea to ssh into the sshuser machine

Now switching to the .ssh directory & we have authorized_keys file, now that interesting 

I try to switch user back to sshuser & i can create an id_rsa & id_rsa.pub file on sshuser machine & use it to pivot 

![image](https://user-images.githubusercontent.com/64267672/168521628-ccf8376e-22be-4fbc-99e4-87de4cff50a6.png)

We can go to the sshuser .ssh directory now where we have our newly created id_rsa files, copy the id_rsa.pub file to root user authorized_keys then use it to connect back via ssh as sshuer & we have our flag 

![image](https://user-images.githubusercontent.com/64267672/168521945-84411e08-003f-4e53-8f2a-e990aa2a37c4.png)

Here is the flag to the challenge viola 

![image](https://user-images.githubusercontent.com/64267672/168522025-ad21a59c-1019-42d5-a8b8-44221c21401e.png)













