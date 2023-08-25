### **CON CREDS**
Per ottenere tutti gli User di un dominio ==avendo un account a disposizione==
```bash
impacket-GetADUsers -all <DOMINIO>/<USER>:<PASS>
```
![[Pasted image 20230825105056.png]]
Si puo usare anche ldapsearch, in modo da cercare gli user anche in altri domini se si hanno trust
```bash
ldapsearch -H ldap://<IP> -D "<USER>k@<DOMAIN DELL'USER>" -w <PASS> -b ',DC=<NOME DOMAIN DA VEDERE>,DC=<.COSE DEL DOMANIN (senza .)>' "(&(objectCategory=person)(objectClass=user))"
```
### **SENZA CREDS**
Usando smb e senza avere creds
```bash
crackmapexec smb <IP> --users
```
Ha SAMRPC attivo e quindi potrò enumerare gli user
![[Pasted image 20230810163553.png]]
Ho ottenuto anche una password trovata nella descrizione negli user

Puoi anche ottenere la policy della password per provare del bruteforce
```bash
crackmapexec smb <IP> --pass-pol
```
![[Pasted image 20230810164319.png]]
Potresti usare anche enu4linux ma non ho voglia