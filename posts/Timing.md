
<img width="1062" alt="Screenshot 2022-03-04 at 16 56 05" src="https://user-images.githubusercontent.com/64267672/156796474-ff946294-6dca-41db-9af4-411e3a72750a.png">

Timing - This is a machine started with a webpage I could get access into by guessing the password (aaron : aaron), then I observed the request with burp & added a role parameter to get admin access of the web page which had an upload feature, i could check the code of different endpoints by exploiting local file inclusion (lfi) & noticed how the filtering is done which is mainly, files are being stored in “images/uploads/“, has special md5 hash attached to form filename, also time uploaded & image size. So I uploaded a php shell with .jpg extension with help of simple script I put together I bypass the filter to find the exact file when after uploading, after getting webshell, I noticed I could read some file & saw a backup zip file I also downloaded through lfi, unzipped the backup file & noticed a hidden .git folder and there I can find password to Aaron user to login ssh. I escalated privileges by exploiting weak file permission(file runs as root), I made ssh file locally & uploaded to attacker machine, then used the key to ssh as root user.


So staring with nmap scan;

![image](https://user-images.githubusercontent.com/64267672/156797208-53187743-16b9-4747-99f7-348f141475c0.png)

i see port 80 is open so i am taking a look at it;

<img width="662" alt="Screenshot 2022-03-01 at 12 03 24" src="https://user-images.githubusercontent.com/64267672/156797498-f2ee51a3-af74-4c23-a6a1-aca20f172ba8.png">

Seeing we are faced with a login page, we can spend time bruteforcing for valid login creds or better still enumerate further;

Fuzzing -

<img width="560" alt="Screenshot 2022-03-04 at 17 09 16" src="https://user-images.githubusercontent.com/64267672/156798552-6cca2397-852c-4309-84bf-fad3e6fd2545.png">

image.php stoodout for me. whenever i access any .php file i get redirected back to the login page, but when i visit image.php i get a blank page

<img width="679" alt="Screenshot 2022-03-04 at 17 16 48" src="https://user-images.githubusercontent.com/64267672/156799614-235c1958-f819-4477-be27-624f5b28b2ae.png">

now i try to test for lfi but i think this page accept some get or post parameter because when we upload any php shell through images it's also need parameter and if we don't pass any they give us blank page like this.

so i try to fuzz what parameter takes & reads the /etc/passwd file

<img width="489" alt="Screenshot 2022-03-04 at 17 35 29" src="https://user-images.githubusercontent.com/64267672/156802644-65d7a428-8564-4fc1-a1b9-e3b20794220c.png">

but when i visit the page i get some error message

<img width="534" alt="Screenshot 2022-03-05 at 04 21 50" src="https://user-images.githubusercontent.com/64267672/156865867-3d440a82-29ba-43ca-bd8b-1744faf8263a.png">

i suspect some filtering is going on, now bypass time;

![image](https://user-images.githubusercontent.com/64267672/156866035-faafa031-a0db-4f63-abf2-0c6f8d11c7eb.png)

i'm able to bypass filtering by returning base64 output back out,

now i can try to read sorce code of php files;

![image](https://user-images.githubusercontent.com/64267672/156866106-42a90ce0-e558-43d1-8a9e-6f47922f3dac.png) 

here i can see there's a check on me if i'm admin user, to be able to upload files, the files uploaded are stored in /images/uploads also ebeing executed my a privileged user hence 0777.

this php code appends a unique md5 hash to it's file name as well as checking for the time it gets uploaded, file extention(jpg), ok noted.

![image](https://user-images.githubusercontent.com/64267672/156866499-eab961eb-ca36-4fe8-9941-99a4854bfb16.png)

this code checks if i am admin user with role permision (1) to be able to access & upload files


![image](https://user-images.githubusercontent.com/64267672/156866624-2f787ee7-6585-4053-938c-abd7f515d7d7.png)

this code checks for the session id of user & sets a time limit

![image](https://user-images.githubusercontent.com/64267672/156866775-c662d268-ecb0-4cfa-ba23-131b0b3e8bc2.png)

now we find some creds reading the database config file. i try it on http login page & nothing worked, aaron : aaron worked as default user login details (/etc/passwd file details)

![image](https://user-images.githubusercontent.com/64267672/156866815-776a3676-538c-46ac-849a-de4afaa7520e.png)

now we have access but as user2, now the understanding of the php source code comes in.

This is the default request when editing/updating info on the web app

![image](https://user-images.githubusercontent.com/64267672/156866860-af4bd614-125f-41e5-85ef-7f515abb3f14.png)

but the php code checks if role != 1, so i can add myself role parameter in request

![image](https://user-images.githubusercontent.com/64267672/156866900-9ea43061-b34c-4990-99aa-bbaff94f6626.png)

and i have admin panel which means i become admin;

![image](https://user-images.githubusercontent.com/64267672/156866923-dde280de-9774-443d-a8a2-883812748f6b.png)

now i upload a php shell but it doen't take jpg so i renamed file to .jpg & made a php script to also check & find potential file name after giving it the file name we uploaded as and unix timestamp;

![image](https://user-images.githubusercontent.com/64267672/156867076-d9c19c35-3414-4482-821d-91c849796d0f.png)

we change the content-type to make it work;

![image](https://user-images.githubusercontent.com/64267672/156867109-6f51dc38-3c29-4acf-b5f1-cb9f11066844.png)

uploaded shell successfully but where & what is it saved as? 

the php code translates that uploads are stored in the /images/uploads/ dir

<img width="482" alt="Screenshot 2022-03-05 at 05 13 09" src="https://user-images.githubusercontent.com/64267672/156867222-6c48b50b-385c-4481-923b-4969d767b534.png">


but we get forbidden when we vist the dir mm, now we use the php script. here's how it works;
upload file, take the time in response & convert to unix timestamp(cyberchef), update php script with file name & unix quivalent time stamp

![image](https://user-images.githubusercontent.com/64267672/156867297-ef7fdb28-b9fb-4279-80ba-4e8515795f86.png)


after finding the file name of shell/file with our php script, we can then locate the file from the /images/uploads/ dir

![image](https://user-images.githubusercontent.com/64267672/156867380-d6a7b8a9-6572-4c7a-a05f-1f51abb6738b.png)

we have a shell now, we can enumerate & we'll find a zip file in system but we can read the file, 
http://10.10.11.135/image.php?img=php://filter/convert.base64-encode/resource=/opt/source-files-backup.zip

we get a base64 output, i found a website to convert base64 output to file(base64 guru)

<img width="788" alt="Screenshot 2022-03-05 at 05 23 33" src="https://user-images.githubusercontent.com/64267672/156867529-7d73ef91-fa09-4c51-8d15-9892ed4e8d70.png">

contents of zip file;

![image](https://user-images.githubusercontent.com/64267672/156867561-b403e463-367a-43c1-a9f8-73aaeff7fc74.png)

there's a hidden .git folder, i find a /logs dir inside the .git folder here i found 2 user creds, i tried the first creds for ssh as aaron user & it worked correctly.

![image](https://user-images.githubusercontent.com/64267672/156867659-48a236f2-2db3-4a68-8ba9-6a0560273627.png)

ssh access as aaron user;

![image](https://user-images.githubusercontent.com/64267672/156867827-517992ae-b680-414c-bef7-d83c9ea5bc14.png)

now we have user.txt and time to root now.

![image](https://user-images.githubusercontent.com/64267672/156867854-98aa34e9-79af-427f-9478-6c8a5b83eabd.png)

running sudo -l reveals the binary /usr/bin/netutils can be used as aaron user but with sudo permission;

![image](https://user-images.githubusercontent.com/64267672/156867921-691e4c43-c5f6-46df-acd8-df97a3ced64f.png)

when i upload a file using the /usr/bin/netutils binary (this case roottest) i see it has the root user file permission automatically 

<img width="566" alt="Screenshot 2022-03-02 at 02 34 08" src="https://user-images.githubusercontent.com/64267672/156867937-5759511b-bd42-41d0-b039-547e751c44e4.png">

now i think i can make ssh-key locally, make a copy or rename .pub key, upload & then use it to access the root user cus file permission.

![image](https://user-images.githubusercontent.com/64267672/156867999-a75179dd-6073-4a1b-b836-ed90689afe79.png)

now i link the potential ssh key with root user authorized keys;
ln -s /root/.ssh/authorized_keys keys

<img width="550" alt="Screenshot 2022-03-02 at 02 50 03" src="https://user-images.githubusercontent.com/64267672/156868231-b98332e6-a81c-41d7-8c2a-540d246fde1e.png">

now i upload the ssh key after linking it, notice it doesn't create a new file after.

<img width="861" alt="Screenshot 2022-03-02 at 02 53 29" src="https://user-images.githubusercontent.com/64267672/156868284-1e3a6369-167f-44b3-8783-b670bc274363.png">

ssh as root user & we have root access on this machine. 




###RESOURCES;

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion#wrapper-phpfilter


https://base64.guru/converter/decode/file


Webshell - https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php


 ```Php script to locate file - 

// save in a file name time.php //

<?php
$upload_dir = "images/uploads/";
$file = "cmd.jpg";

while(true){
    $file_name = md5('$file_hash'. 1646184162) . '_' . $file;
    $target_file = $upload_dir . $file_name;
    echo $file_name;
    echo PHP_EOL;
    sleep(1);
}

```




