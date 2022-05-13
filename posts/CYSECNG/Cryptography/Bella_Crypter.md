Bella Crypter

![image](https://user-images.githubusercontent.com/64267672/168384699-b1a5c70d-255d-43b8-bdcf-3a9e9c652a69.png)

We can copy & paste this random string (hash) into google.com & we will see so many linked resources using the same hash, most are decoded already.

We also can use hashcat tool with this command 'hashcat -m 3200 hash.txt rockyou.txt ' where hash.txt is where our hash is being stored & rockyou.txt contains the list of passwords to iterate through & locate a valid result from

of course i used google lol. I have the same hash & decoded string linked below

https://lightorithm.gitbook.io/searchlight/crack-the-hash

now we know our decoded string is "bleh"

![image](https://user-images.githubusercontent.com/64267672/168385298-6f82b2ee-a844-4d45-8bc7-eccf29c31054.png)


Our flag is CYSEC{bleh}. Hope you enjoyed that :)
