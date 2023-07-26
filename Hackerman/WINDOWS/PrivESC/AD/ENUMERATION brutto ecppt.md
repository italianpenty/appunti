Moduli usati:
	-PowerView
	-Ad Powershell module

### **IDENTIFICAZIONE MACCHINE NEL NETWORK**
Per identificare altre macchine all'interno del network si può effettuare un reverse lookup via LDAP
![[Pasted image 20230524152213.png]]
### **SCOPRIRE LE GROUPS POLICIES ALL'INTERNO DEL DOMAIN**
Con PowerView lanciare il seguente comando
![[Pasted image 20230524152501.png]]

### **USER HUNTING**
Usando powerview
![[Pasted image 20230524153006.png]]
O senza
![[Pasted image 20230524152853.png]]

### **GPO ENUMERATION**
Usando PowerView
Usa Find-GPOLocation
Per trovare gli admin di uno specifico computer
![[Pasted image 20230524160618.png]]
Per identificare quali utenti appartengono al gruppo "local admin"
![[Pasted image 20230524160847.png]]


