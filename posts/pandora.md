We would be attacking the HackTheBox Pandora Machine

![image](https://user-images.githubusercontent.com/64267672/162553886-1744024a-ab4d-48a8-8838-274d9d7c3dd9.png)


Definitely starting with an nmap scan 

![image](https://user-images.githubusercontent.com/64267672/162553445-ff299b13-3aaf-4de0-9d21-cfe0ffaedaaa.png)

Webpage on port 80 looks quite empty, safe some email addresses.

![image](https://user-images.githubusercontent.com/64267672/162553468-b283ac0e-623e-4511-91cf-c0e2cc075be1.png)

Now we do a full nmap scan & udp port scan 

![image](https://user-images.githubusercontent.com/64267672/162553499-962ec028-525c-4ead-bb46-f66e87b76b96.png)

we find snmp service running on Udp port 161 open;

![image](https://user-images.githubusercontent.com/64267672/162553523-cbc20011-551c-4c85-aaf0-8ff0d6a42d9a.png)


Enumerating udp port 161 with snmpwalk tool;

![image](https://user-images.githubusercontent.com/64267672/162553580-029cc851-24fa-4ed3-b5d4-d5c540ab8862.png)

We can get username & password from snmp

now we use found creds to login ssh as daniel user

![image](https://user-images.githubusercontent.com/64267672/162553612-5b618671-f068-42ab-b6ce-ca3c024309c8.png)

Password here;

![image](https://user-images.githubusercontent.com/64267672/162553683-a917a273-36eb-41ca-9632-96879ebda457.png)

We get ssh access & we find linpeas to escalate our privileges as daniel to root user;

![image](https://user-images.githubusercontent.com/64267672/162553714-7761a2fb-500e-4037-90af-990dcd1143c0.png)

We find the system vulnurable to pwnkit CVE-2021-4034;

changing our directory to the /tmp/dm0220 we find the pwnkit exploit lying around

Exploiting pwnkit CVE-2021-4034 to get root!

![image](https://user-images.githubusercontent.com/64267672/162553820-7de3beee-9035-4c3e-a068-f7025128398e.png)

We become root now & can read both root.txt from root user & user.txt from the user matt


###RESOURCES

https://app.hackthebox.com/machines/Pandora


