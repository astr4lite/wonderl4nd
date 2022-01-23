# Here Are Notes For My WriteUp

## Enumeration
nmap scan:
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
dirb scan:
```
---- Scanning URL: http://10.10.2.87/ ----
==> DIRECTORY: http://10.10.2.87/img/                                             
+ http://10.10.2.87/index.html (CODE:301|SIZE:0)                                  
==> DIRECTORY: http://10.10.2.87/r/    
```

### [port 80] - website
title says "Follow the White Rabbit.", there is also a picture of a white rabbit on the site
![[Pasted image 20211022000513.png]]

during the dirb scan we find a directory "r", now when we go to this directory we are confronted with this message:
```
# Keep Going.

"Would you tell me, please, which way I ought to go from here?"
```
![[Pasted image 20211022001051.png]]
now, knowing what we know with all the rabbit stuffs we can assume there might be another directory following the word rabbit, and sure enough when we direct to "/r/a" it continues this message:
```
# Keep Going.

"That depends a good deal on where you want to get to," said the Cat.
```
![[Pasted image 20211022001227.png]]
now im not going to waste time in this so we'll just skip to the part where we get to "/r/a/b/b/i/t" where we find this:
![[Pasted image 20211022001333.png]]

## Initial Foothold
---
now, inspecting the source code we find an ssh username and password for the user "alice" 
![[Pasted image 20211022001558.png]]
```
alice:HowDothTheLittleCrocodileImproveHisShiningTail
```
we can assume this is an ssh login as it is the only other service running on this machine, and sure enough... it works!
![[Pasted image 20211022001830.png]]

![[Pasted image 20211022002113.png]]
listing alice's directory we find two iregular things, for one the root flag is stored on the user's profile, yet no user flag is to be found, odd. the next iregular thing would be the python script named "walrus_and_the_carpenter.py"


now, lets start some priviledge escalatin !!

## Priviledge Escalation
i love this machine as it requires alot of horizontal movement, and features some cool privesc, not just a kernel exploit here and there (atleast, not at the beginning, tehe)

### Moving Laterally (alice -> rabbit)
first thing i usually do when i have a password for a user is to run "sudo -l" to see if we are able to run anything as root or a more priviledged user. running this here we find out we can run the python script we found earlier as the user "rabbit"
![[Pasted image 20211022002946.png]]

now looking into the script, we see that it imports the "random" module and prints 10 random lines from the "poem" variable:
```python
import random
poem = "long fucking poem lulull"

for i in range(10):
    line = random.choice(poem.split("\n"))
```
**NOTE: THE POEM VARIABLE HAS BEEN SHORTENED AS TO NOT MAKE THIS SCRIPT TAKE UP HALF OF THE PAGE**

now you may be thinking to yourself, welp this script cant help us at all, its IRON CLAD!! welp, you'd be wrong !!!! 

before we start writing our own script, let me give you a brief lesson in how python accesses modules. now when you import a module python first, before looking for it elsewhere checks locally, within the folder that the script is stored in to see if there is a python file with the same name. 

knowing this we can now go ahead and intercept the search for the "random" module by writing our own "random.py" script, using the "choice" function to spawn a shell, as this is what function the script is accessing. here is what our "random.py" should look like

```
import pty

def choice(x):
    pty.spawn("/bin/bash")
```
remember and add the "(x)" as the script provides an input and we dont want to cause any unwanted errors, and we dont need to return anything because we will have shell lul (technically we could set a keypress to return and exit the program if we need a way out but i cba writing that in lulu :p)

now for a sanity check lets run the code on the interpreter just to check pty will work on this machine:
![[Pasted image 20211022004354.png]]
as we can see, it works perfect :)

now running 
```
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```
we should now get a shell as rabbit!
```
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
rabbit@wonderland:~$
```

now moving to /home/rabbit we find a perculiar file 
![[Pasted image 20211022004932.png]]

---
### Moving Laterally Once More (rabbit -> hatter)
looking at this file's permisions we see that it is a SUID binary !!!! woow, now how this is going to work is verry similar to what we just did with random.py, only now with bash !!!!!
![[Pasted image 20211022005141.png]]

as we can see this is an executable, not a script so we cant just cat it out like normal... or can we?

![[Pasted image 20211022005347.png]]
catting out the executable shows a bunch of gibberish but around the middle we can see some text !!!!! normally we would run strings to see this more clearly but this machine doesnt seem to have it installed :(

looking at the text we can read we see a possible attack vector... date!!!!!

now you may also see the echo command being used, but we cannot use this as an attack vector as this uses a full path to the echo file, however, date does not. and we can use this to our advantage.

#### SUID TUTORIAL FOR NEWBS
#### #1 - FIND ATTACK VECTOR
done :)
#### #2 - MAKE UR SCRIPT
now we have our attack vector "date", we can now create a fake "date" script and make the "teaParty" script run our fake one and give us a shell, ez-pz!!! write this script and save it as "date"
```bash
#!/bin/bash

bash
```

#### #3 - MAKE IT ABL 2 RUN
now run chmod +x on our script to make it executable
```bash
chmod +x /home/rabbit/date
```

#### #4 - ADD THE SCRIPT'S FOLDER TO THE START OF THE $PATH VARIABLE SO THE EXECUTABLE SEE'S OUR MALICIOUS SCRIPT FIRST
move to /home/rabbit/date and run this command
```bash
export PATH=$(pwd):$PATH
```

#### #5 - HOPE FOR THE BEST
now for the moment of truth, run the teaParty script and you should hopefully have a shell!!!!!
![[Pasted image 20211022010818.png]]

and boom !!! we have sucessfully privesc'd 2 accounts. now to get to root, hehe :3

we can start by moving to /home/hatter where we find an ssh password for hatter, deserved after having to deal with 2 variations of shitty shells heheh..
![[Pasted image 20211022011257.png]]

### The Final Beast - (hatter -> root)
Congrats, we are now on the last stage, and probably the most fluid method of them all, the few before this were layed pretty much right infront of us, but this is more just up to what you know.

now personally the method i went for was setuid capabilities found in perl, now this, as far as i know isnt very common. or atleast very basically known, but my friend stumbled upon it while running privesc enum scripts and looking at gtfobins

so run:
```
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```
and hey presto!! we have root!
![[Pasted image 20211022012719.png]]

capabilities seems to be really powerful, so im going to research them a bit more after this hahah

## Finding user.txt
now we know where root.txt is, but we never saw user.txt anywhere..

```
find / -type f -name user.txt 2>/dev/null
```
![[Pasted image 20211022013113.png]]

hmmm... it seems root and user switched places !!!! well, this is very odd heheh.. oh well, not the matter, we can now access both!!!
![[Pasted image 20211022013217.png]]

![[Pasted image 20211022013259.png]]

boom, one step closer to 0xA, heheh


