### **SUBNET**
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

### **USER**
Enumero gli user presenti su ogni macchine
```bash
crackmapexec smb 192.168.56.11 --users
```
Ha SAMRPC attivo e quindi potrò enumerare gli user
![[Pasted image 20230810163553.png]]
Ho ottenuto anche una password trovata nella dexs