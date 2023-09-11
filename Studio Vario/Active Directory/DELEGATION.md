![[Pasted image 20230911095916.png]]
La delegation permette al web server di interpretare l'utente per accedere ai suoi file, facendo credere al file server che sia l'utente ad effettuare le azioni.
L'abilità di relay delle credenziali può essere data ad un account che abbia almeni un SPN.
I tipi di delegation ad oggi sono tre:
- Uncostrained Delegation
- Constrained Delegation
- Resource Based Constrained Delegation
### **Uncostrained Delegation**
Con questo tipo di delegation il server o il service account a cui viene garantita la delegazione può impersonare l'utente per qualsiasi servizio o host.
![[Pasted image 20230911102947.png]]
### **Constrained Delegation**
Il delegato avrà una lista di utenti di cui ha la delega e una lista di servizi o server a cui può far accesso con determinate deleghe
![[Pasted image 20230911103431.png]]
### **Resource Based Constrained Delegation**
Come la Constrained Delegation, ma in questo caso la lista degli account che possono es