### **BYPASS POWERSHELL POLOCIES**
```powershell
powershell -ep bypass
whoami /all | Select-String -Pattern "jareth" -Context 2,0
```powershell
import-module .\PowerView.ps1
```
### **ENUMERATE DOMAIN USERS**
```powershell
Get-NetUser | select cn
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png)

Chi appartiene al gruppo "admin"

![[Pasted image 20230503131149.png]]

### **CONTROLLARE GLI SHARE PRESENTI**

![[Pasted image 20230503131251.png]]


### **CHEATSHEET**
https://gist.githubusercontent.com/HarmJ0y/184f9822b195c52dd50c379ed3117993/raw/e5e30c942adb2347917563ef0dafa7054882535a/PowerView-3.0-tricks.ps1