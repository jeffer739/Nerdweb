Easy Web

![image](https://user-images.githubusercontent.com/64267672/168396721-f6feff4c-c709-4821-a4ce-bf66fe34f335.png)

Visiting the web page 

![image](https://user-images.githubusercontent.com/64267672/168396813-322d3813-30bc-47b8-a44f-e4e1117ae1ea.png)

We have a lot going on already thanks to my 1337 l33t fwens haha

![image](https://user-images.githubusercontent.com/64267672/168397551-9b70a0d3-ba54-4dd9-a9e4-abb39b588591.png)

Now we can view the page source of the web page, we can also observe the comments in the code 

!-- /api?id=source --> This one being the standout comment

When we try to visit this /api endpoint with the id parameter & "source" value, we are able to download the source code of the web page 

now let's view the code 

![image](https://user-images.githubusercontent.com/64267672/168398364-dd51b55b-dd7b-46c7-a3cb-6bb804f92b5f.png)

This code checks the body of every http requests we make to see if we give the id parameter the "gimme-flag" value else it tells us "I don't what you speak of, h4x0r!"

So with this idea i decided to use curl to make the request & get the flag for this web challenge ez

![image](https://user-images.githubusercontent.com/64267672/168398766-1297e63a-f1f0-4745-8938-f972f6f6b588.png)


And we have our flag for this challenge :) I hope you enjoyed this one as i did 😉
