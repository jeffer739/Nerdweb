
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

