![[Pasted image 20230911095916.png]]
La delegation permette al web server di interpretare l'utente per accedere ai suoi file, facendo credere al file server che sia l'utente ad effettuare le azioni.
L'abilità di relay delle credenziali può essere data ad un account che abbia almeni un SPN.
I tipi di delegation ad oggi sono tre:
- Uncostrained Delegation
- Constrained Delegation
- Resource Based Constrained Delegation
### **Uncostrained Delegation**
Con questo tipo di delegation il server o il service account a cui viene garantita la delegazione può impersonare l'utente per qualsiasi servizio o host.