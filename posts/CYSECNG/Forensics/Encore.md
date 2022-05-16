Encore

![image](https://user-images.githubusercontent.com/64267672/168539001-5e6e229f-4d3e-464f-9eba-6f9df03ba3cf.png)

After downloading the attachment, we can use binwalk tool to investigate & extract hidden embedments

![image](https://user-images.githubusercontent.com/64267672/168539827-99cdbfc5-39a0-49ee-990e-e5f79084a7ee.png)

Now we can switch to the newly discorverd directory, here we find a new text file & a zip file

![image](https://user-images.githubusercontent.com/64267672/168540289-b9fadb8a-9358-4241-9041-020c138629e2.png)

Now reading the content of the text file & we can see a hint, some credential, potential password but it has a lot of weird line spacing 

![image](https://user-images.githubusercontent.com/64267672/168540653-fba62a11-5e04-4fea-9b3e-fd092803de06.png)

now i think we can use stegsnow tool here with found creds

stegsnow -C -p "Pass_word$323" '==.txt'

And we get our flag for this challenge
