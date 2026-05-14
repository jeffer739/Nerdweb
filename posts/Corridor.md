Hello Long time no Hack no Blog but we back now. Let's get started with Corridor challenge from Webverse.



*Corridor challenge from Webverse simulates a website called Ridgeline Press. This website at first view look very static but upon further digging and investigation i find out the website is vulnurable to a Local File Inclusion Vulnurability that allows arbitrary file access on the server. Once exploited we can get access to sensitive files and informations and ultimately our flag.*



<img width="1243" height="946" alt="image" src="https://github.com/user-attachments/assets/b7b5e64b-2aab-461d-b2ec-0713606f6e65" />

Here we have our dashboard after starting up the the machine and receiving our IP address to attack. 

We have a DNS problem and to fix we have to resolve the IP address to the hostname (ridgelinepress.co) 

<img width="1261" height="932" alt="image" src="https://github.com/user-attachments/assets/99cf6d6b-81b5-4bd6-bdb4-a96dc5ffdeca" />

Here after we resolve the DNS problem (It's always DNS) 

<img width="835" height="313" alt="image" src="https://github.com/user-attachments/assets/8d051dc5-746b-4c09-ae00-c833720ab3e6" />

New Look web page after DNS resolve.

 <img width="1275" height="945" alt="image" src="https://github.com/user-attachments/assets/c66d20b2-7637-4f75-adbb-dc0584cd4f54" />

Browsing through the web pages and features i notice the many usernames as they could be useful, i also try to observe the robots.txt and we have an interesting find 

<img width="1268" height="525" alt="image" src="https://github.com/user-attachments/assets/766d18fa-ae3e-42f4-84bb-3912b8020f5a" />

Here we see a hidden notes.html file inaccessible to users. Now i try to access the file but i have to browse through the web pagess to see how the it call up resources on the webpage and i found an interesting parameter *slug=tidewater-by-anna-mills.html* 

<img width="1918" height="855" alt="image" src="https://github.com/user-attachments/assets/79d910e9-0c7d-4dbe-a9f8-b1f715792d43" />

Now i see i can access a file from the web server called **tidewater-by-anna-mills.html**

This is a strong indicator of a potential Local File Inclusion, user input is not properly validated.

But i need to access the interesting notes.html file instead and i have just one job

<img width="1918" height="862" alt="image" src="https://github.com/user-attachments/assets/c71d7eaf-7adb-49e7-895e-baaf0beb0bf0" />

This is the result in browser

<img width="1266" height="1002" alt="image" src="https://github.com/user-attachments/assets/65f13c2d-8736-4418-8eaf-de89ea25c908" />

Also testing for more LFI POC. I get access to /etc/passwd here

<img width="1910" height="735" alt="image" src="https://github.com/user-attachments/assets/9cfce40a-63d4-4ce3-8632-f417adb50d52" />

How it look in browser 

<img width="1261" height="958" alt="image" src="https://github.com/user-attachments/assets/797393ad-d9b8-4ad4-ad05-0e1c1ee907db" />

Calling back the informations from notes.html we have our flag in the home directory of the Mike user. Now we have our flag

<img width="1920" height="863" alt="image" src="https://github.com/user-attachments/assets/2a56ab25-b6f6-4f68-9f95-e753070bdf21" />

Browser look

<img width="1262" height="636" alt="image" src="https://github.com/user-attachments/assets/a64fed41-78c1-4e7a-8a76-ba3d4b439a3c" />
