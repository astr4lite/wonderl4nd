nmap scan
```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

dirb scan
```
┌──(lite㉿computer)-[~]
└─$ dirb http://10.10.2.87/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Oct 21 14:13:09 2021
URL_BASE: http://10.10.2.87/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.2.87/ ----
==> DIRECTORY: http://10.10.2.87/img/                                             
+ http://10.10.2.87/index.html (CODE:301|SIZE:0)                                  
==> DIRECTORY: http://10.10.2.87/r/      
```
/r tells us to keep going, and with a bit of guessing i assume its telling us to spell out
"r/a/b/b/i/t" because why wouldnt it. lol

**SIDENOTE: i was in a call with friends as we were doing the box together, one had done some stego on the image on the index page and found out that when extracted said "follow the r a b b i t"

the page looks similar to the first but on further inspection, the source code on http://$IP/r/a/b/b/i/t/ shows alice ssh credentials

```
alice:HowDothTheLittleCrocodileImproveHisShiningTail
```


-- continue laterhttps://tryhackme.com/room/wonderland https://gtfobins.github.io/gtfobins/perl/#capabilities