Writeup for Tryhackme Vulversity ctf challenge.



![vulnversityinfo](https://user-images.githubusercontent.com/64267672/125898695-f9675392-6613-46e3-a863-3854441856a4.png)




Starting with an nmap scan foinformation gathering;
![vulnversitynmap](https://user-images.githubusercontent.com/64267672/125898736-ce5c5152-0a60-499b-8697-f8995f410a56.png)

Looking at the web server on port 3333 & we have this;
![vulnversityweb](https://user-images.githubusercontent.com/64267672/125899869-bb9d8ff0-7dc2-4044-90bf-daa20a41838d.png)

Now bruteforcing for interesting directories some;
![vulnversitybuster](https://user-images.githubusercontent.com/64267672/125899970-b95c697f-7942-4e34-999b-2339170d23d2.png)
we see that there's a hidden '/uploads' directory indside of '/internal' directory all in the web server on port 3333

visiting the /internal directory reveals to us the upload feature of the website
![vulnversityinternal](https://user-images.githubusercontent.com/64267672/125900826-db85ce15-de0b-4b34-9787-21d12c21c5f5.png)


Most time the upload feature of a website can be vulnurable & an attacker can try to upload malicious content into the server just like I'm gonna do. Now i can try to upload a reverse shell;
![vulnversityupload](https://user-images.githubusercontent.com/64267672/125900859-bfcc73fb-b018-4113-9c6e-76271ef6ab2e.png)
(Before upload)
Ok so we get our php reverse shell ready with the listening ip address & port number (attacker's);
![vulnversityphpshell](https://user-images.githubusercontent.com/64267672/125902181-25e10dc9-c785-4529-8f60-ed78cbd19176.png)

ok we can start testing the upload feature with whatever file extentions you prefer, but i started with a .php file extention but .php file extention is being filtered apparently;
![vulnversityphpfilter](https://user-images.githubusercontent.com/64267672/125902716-2e26df2e-af66-49c2-a4dc-17decaf0109d.png)
But GOODNEWS!! This can be bypassed. 
![vulnversityphpfile](https://user-images.githubusercontent.com/64267672/125903288-51289c6c-49fb-4cfc-a37a-131321557c9e.png)

ok so making a list of the file extentions i can bruteforce & see which works ok;
![vulnversityburpfile](https://user-images.githubusercontent.com/64267672/125903670-2da7cd4a-8ed0-4b45-a865-120d90763358.png)
![vulnversityburp](https://user-images.githubusercontent.com/64267672/125903779-ee8c47fe-5ed9-4c7d-a0dd-b22c218dadf9.png)

Now we have a result & 1 lucky hit;
![vulnversityintrud](https://user-images.githubusercontent.com/64267672/125903949-5833a289-3298-4f09-9829-7f7b37bfbb2c.png)


so we now what file extention to save our reverse shell now, so we try to upload malicios file but this time saving it as .phtml;
![vulnversityphpsuccess](https://user-images.githubusercontent.com/64267672/125904448-4d7fa0cc-276c-4725-87df-49f6a6c86b52.png)

And here we go!!
we can find our file in /uploads directory;
![vulnversityfileupload](https://user-images.githubusercontent.com/64267672/125904738-c2c6ddb5-e50c-4902-a00d-48793a1b8caf.png)

Now we can setup our netcat listener to listen on the specified port we set on our reverse shell script, mine being port 1337;
![vulnversitynetcat](https://user-images.githubusercontent.com/64267672/125905190-a7dc5352-eb12-449d-895e-24a60502cfe9.png)

BOOM!! we have a shell as www-data user;
![vulnversityshell](https://user-images.githubusercontent.com/64267672/125905454-9ec6b6f9-0dd4-40b9-a31b-484914bc3359.png)

now to escalate to higher privilege (root);



