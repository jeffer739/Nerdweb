![image](https://user-images.githubusercontent.com/64267672/168320427-ee185a72-4b50-45e0-bc1a-6fc6463246c9.png)

Here we are given a go binary to reverse & get the flag,

I downloaded the binary, ran the binary but it asks for username & also password 

![image](https://user-images.githubusercontent.com/64267672/168320902-004ebea0-2407-4cc0-a45b-fef3b5901e48.png)
 
 But here we don't know any password. i ran strings command on the binary,
 
 ![image](https://user-images.githubusercontent.com/64267672/168322438-684e675b-c1e1-475a-ac2b-7aa459d4bbbb.png)

confirmation binary is written in golang

And now we can see the binary is also packed using upx 

![image](https://user-images.githubusercontent.com/64267672/168373834-c4c9febb-b5b0-4c4e-9808-5903316dbb76.png)

Now reaserching about upx & we can see it is the ultimate packer for executables https://upx.github.io/

Now we just have to unpack using upx also 

![image](https://user-images.githubusercontent.com/64267672/168374271-fdb5a586-e238-4ce6-bda0-97769ed0fc5a.png)

Now we see the ration, got decompressed 42.70% 

Now running the strings command again but this time grepping for the flag keyword "CYSEC" 

![image](https://user-images.githubusercontent.com/64267672/168374587-1105aa1d-574b-4d72-979a-e177a6a20219.png)


Now we have our flag & we can submit :)

Hope you enjoyed that, i did 😉
