### **REGISTRY**
Cerca delle password nel registro
```powershell
reg query HKLM /f password /t REG_SZ /s
```
### **SAVED CREDS**
```powershell
cmdkey /list
```
