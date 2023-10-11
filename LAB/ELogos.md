### **PORT SCANNING**
```bash
nmap -sT -T5 --min-rate=10000 -p- 192.168.86.129
```
![[Pasted image 20231011153211.png]]

### **PATH 1 - FTP**
Go to http://elogos.ctf/about-us and take note of the listed username (email)
![[Pasted image 20231011153524.png]]

After that launch a brute force with hydra o