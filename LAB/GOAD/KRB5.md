### **SETUP**
Modifico il file /etc/krb5.conf per setuppare i realm![[Pasted image 20230810124518.png]]

### **ENUM USER**
Trovo username appartenenti alle varie foreste (lista fatta usando gli schemi )
```bash
kerbrute userenum -d essos.local --dc 192.168.56.12 user.txt
```
![[Pasted image 20230810125834.png]]
![[Pasted image 20230810154111.png]]
![[Pasted image 20230810154242.png]]





### **TGT BruteForce**
Non ho ben capito come venga ottenuta la password 'horse', force facendo un bruteforce è possibile
