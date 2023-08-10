Inizio enumerando l'intera subnet con cme alla ricerca du samba
```bash
crackmapexec smb 192.168.56.0/24
```
![[Pasted image 20230810122313.png]]
Ora trovo gli ip dei DC per configurare /etc/hosts e capire la topologia
```bash
|1|nslookup -type=srv _ldap._tcp.dc._msdcs.sevenkingdoms.local 192.168.56.10|
```