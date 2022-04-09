Hey we are targeting lampiao machine

Starting with an nmap scan (full scan took a million years to finish, i loked at a writeup to get the full scan result)

![image](https://user-images.githubusercontent.com/64267672/162548377-04c735c4-137b-47cb-b750-523aaee60584.png)


Looking at the webpage on port 1898, we can find 2 users has comments on the page (2 users) quickly

![image](https://user-images.githubusercontent.com/64267672/162544872-cf4b5d71-a1c5-430f-9c50-6eafaa952f24.png)

Try to create an account & we wait for admin approval, which isn't playing nice with us 

looking at /robots.txt we find a lot of stuff on there

![image](https://user-images.githubusercontent.com/64267672/162545077-b9f42ab0-05aa-407b-9f41-1dd69df134fb.png)

visiting different endpoints found from /robots.txt & we find interesting stuff from /CHANGELOG.txt also /web.config

Here we see the cms runs drupal 7, looking around & we can see it's associated with a lot of vulnurabilities, you can get foothold by exploiting drupal 7

What i did here was generate a wordlist using cewl & password spray the found usernames with generated wordlist & i got a valid password used to ssh as Tiago

![image](https://user-images.githubusercontent.com/64267672/162547220-de969b41-715c-4aed-9cef-74f6b8cfb477.png)

Using found valid password to login ssh as tiago user & we can read local.txt

Also checking for sudo permisson shows that the user tiago may not run commands as sudo

![image](https://user-images.githubusercontent.com/64267672/162547286-68548af5-b49b-4030-aeae-474dffd08086.png)


Now getting linpeas & linux exploit suggester on target machine looking for privilege escalation means & there are a lot 

I decided to exploit pwnkit A.K.A CVE-2021-4034 & become root!

![image](https://user-images.githubusercontent.com/64267672/162548147-49e84669-4db7-4516-a709-769838cfed97.png)


####RESOURCES

https://portal.offensive-security.com/proving-grounds/play

