Enjoy FunboxEasyEnum!

Starting with an nmap scan 


![image](https://user-images.githubusercontent.com/64267672/167996061-bf749695-0cdf-41ab-a9a8-55fa925f9f76.png)

looking at the http server at port 80 & we have a default apache webserver, now we can fuzz for hidden directories here

![image](https://user-images.githubusercontent.com/64267672/167997168-c4fb4903-494c-4387-b100-31525763af16.png)


We have quite some results back, after manual inpection mini.php stands out. 

![image](https://user-images.githubusercontent.com/64267672/168005350-8e3979f5-99f9-49e8-8fdc-446d31812fd6.png)


Visiting the mini.php we find a mini shell/ upload feature, we can abuse it & try to upload a reverse shell

![image](https://user-images.githubusercontent.com/64267672/167997562-e1825857-df34-458b-860a-3203eb2a0db5.png)

Uploading a php webshell worked, i tried to connect back to shell on terminal with a php reverse shell

![image](https://user-images.githubusercontent.com/64267672/167997884-a84dc825-fd62-45f4-b768-d87b6107845c.png)


Now i can read local.txt 

![image](https://user-images.githubusercontent.com/64267672/167998324-7bd2eee3-680b-47f9-8ee7-3e4bb0350085.png)


Now unto privilegde escalation 


Visiting the /home directory & we can find couple of users (4)

![image](https://user-images.githubusercontent.com/64267672/168001805-3b815510-f839-443e-a9f6-23ce1d3ba311.png)

We can try to brute force each user for ssh login since we have an open ssh port 22 from our earlier nmap scan, which i did

We have creds for goat user now, we can use creds to login ssh now

We can enumerate everything for Priviledge Escalation vectors, doing that i find out that goat user can run "/usr/bin/mysql" as sudo

Gtfobins to the rescue 

Running sudo mysql -e '\! /bin/sh' gives us a root shell and we are root!

![image](https://user-images.githubusercontent.com/64267672/168003388-ffd64dbf-5f9f-42f3-9ba4-3a6e6b1e8b26.png)



###RESOURCES 

https://portal.offensive-security.com/proving-grounds/play


https://gtfobins.github.io/gtfobins/mysql/#sudo


https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php

