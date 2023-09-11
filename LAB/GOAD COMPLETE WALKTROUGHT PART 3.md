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
