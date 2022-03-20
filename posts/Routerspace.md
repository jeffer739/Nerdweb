![image](https://user-images.githubusercontent.com/64267672/159147316-a1401d89-57cf-4a73-912f-7f2ae9774da8.png)

Attacking Routerpace machine todayy.

Nmap scan 
![image](https://user-images.githubusercontent.com/64267672/159156238-60c8859f-e839-4f96-93d9-c8b92e9046b6.png)


looking at the http port 80, we find a webpage with a link to download an apk file which we download back to local system, run it & proxy it through burpsuite

we download RouterSpace.apk

we can reverse or run through gui which we use here using anbox

we check the app & discover a check status button from the apk 

so we setup proxy

adb shell settings put global http_proxy 192.168.31.208:8080

adb install RouterSpace.apk

<img width="1100" alt="Screenshot 2022-03-20 at 04 51 39" src="https://user-images.githubusercontent.com/64267672/159154069-fe3bcc14-d177-4f8a-9f49-3a2236e3c6ba.png">

observing the request with burpsuite;

<img width="833" alt="Screenshot 2022-03-20 at 09 40 16" src="https://user-images.githubusercontent.com/64267672/159155172-b648e1c3-0fa5-4c23-aa1e-07b57d1ea265.png">

tried some command injection & worked

<img width="771" alt="Screenshot 2022-03-20 at 09 56 01" src="https://user-images.githubusercontent.com/64267672/159155234-063bc6a6-f0af-4c20-a845-49c532a58d9a.png">


i tried different ways to get shell but nothing cus "ip tables" then i looked in .ssh directory for any id_rsa key but nothing, i saw the directory is writeable,

so i generate & upload my public id_rsa key locally & upload to target .ssh directoy.

so generate ssh keys with ssh-keygen 

<img width="548" alt="Screenshot 2022-03-20 at 10 00 03" src="https://user-images.githubusercontent.com/64267672/159155419-3091224f-7d24-4998-af39-830633b471dc.png">

save the content of id_rsa.pub as authorized_keys

<img width="850" alt="Screenshot 2022-03-20 at 09 46 35" src="https://user-images.githubusercontent.com/64267672/159155474-56e2c6fa-0037-4bcf-bad9-dc9535ec5c06.png">



give 700 permissions


<img width="843" alt="Screenshot 2022-03-20 at 09 47 32" src="https://user-images.githubusercontent.com/64267672/159155482-e474ea9f-0825-4021-bf66-507b9b298f8e.png">


now we can ssh with the keys

<img width="1280" alt="Screenshot 2022-03-20 at 10 02 44" src="https://user-images.githubusercontent.com/64267672/159155504-f7866cfe-e280-4fd5-82b1-4f0d94445942.png">

now we have shell access through ssh & here user.txt

now to get root, i noticed through linpeas result target is running vulnurable sudo version 1.8.31 / CVE-2021-3156

did some research & found some known public explot for it 

now transfer exploit to target machine & run it and we are root!

TRANSFER FILE WITH SCP TO TARGET - sudo scp -i /Users/nerdy/.ssh/id_rsa exploit.py paul@10.10.11.148:.


<img width="448" alt="Screenshot 2022-03-20 at 10 22 10" src="https://user-images.githubusercontent.com/64267672/159156000-599e31f0-1512-441c-938b-eb1c0b195f32.png">


###RESOURCES
https://app.hackthebox.com/machines/RouterSpace
https://github.com/worawit/CVE-2021-3156/blob/main/exploit_nss.py


