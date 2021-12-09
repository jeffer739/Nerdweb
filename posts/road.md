We start off with an nmap scan


![image](https://user-images.githubusercontent.com/64267672/145338498-e1adeaa8-6c01-4278-a321-4a77e2b5ba68.png)


Starting enumeration off from port 80 - http

We can see the home page;


![image](https://user-images.githubusercontent.com/64267672/145338666-84ce1e70-789c-4675-9dc6-655171a3891b.png)


We observe we are able to create an account & login

![image](https://user-images.githubusercontent.com/64267672/145338752-42b7863c-2102-438e-bf2e-be921a942e16.png)


And we are logged in as the user we just created;

![image](https://user-images.githubusercontent.com/64267672/145339853-7e823190-e88f-4a7f-93a3-54b310ac532a.png)


Looking at the profile page we see we are able to upload a profile picture BUT as an admin,

![image](https://user-images.githubusercontent.com/64267672/145340191-b861830e-266b-471b-97e4-5dc9845cbac9.png)



So we have to look for a way to be admin;

![image](https://user-images.githubusercontent.com/64267672/145340375-4ff86cc8-134a-47ea-a95e-17d67dcc6da9.png)



![image](https://user-images.githubusercontent.com/64267672/145340814-04eccdec-fd25-4915-9405-ff4ef4f89e4b.png)



![image](https://user-images.githubusercontent.com/64267672/145340945-28f1f77e-6afa-4b8b-ab07-c8aa00bbaa15.png)


We see we can change the admin password by replacing our email address with the admin address provided for us, ^^as shown;

now we can sign out & login with the admin account now


![image](https://user-images.githubusercontent.com/64267672/145343047-e930064d-61f7-49ed-aff5-cb2115cc5ff5.png)


And we have access to test the upload feature & we are def going to try to upload a malicous file to give us shell access

![image](https://user-images.githubusercontent.com/64267672/145343322-92945e85-e397-4977-8a3e-325abb0f896c.png)


Now we able to upload a php file & it got accepted, NO FILTER! (bad)

we look at our page source code & we get some hint possibly to where files like profile images are stored.

![image](https://user-images.githubusercontent.com/64267672/145343717-9cffb5ea-0bba-4cce-b4c6-fa3a5efcf5b1.png)


now we can access our malicious php reverse shell script while we set our listener up waiting for a connection 

![image](https://user-images.githubusercontent.com/64267672/145344558-43a8ec79-e3df-4584-9358-86f58b301ecd.png)


We got a shell as well as prove as user.txt,


We enumerate again to escalate priviledge;

