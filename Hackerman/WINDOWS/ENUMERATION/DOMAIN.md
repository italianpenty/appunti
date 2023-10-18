
*Tools*
- [samratashok/ADModule: Microsoft signed ActiveDirectory PowerShell module (github.com)](https://github.com/samratashok/ADModule)
- [BloodHoundAD/BloodHound: Six Degrees of Domain Admin (github.com)](https://github.com/BloodHoundAD/BloodHound)
- [PowershellMafia/PowerView (github.com)](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
- [tevora-threat/SharpView: C# implementation of harmj0y's PowerView (github.com)](https://github.com/tevora-threat/SharpView)
____
==InviShell >>>==
### **DOMAIN ENUMERATION**
*Get current domain*
```PowerView
Get-Domain
```
![[Pasted image 20230606170600.png]]

```AD_module
Get-ADDomain
```
![[Pasted image 20230606171417.png]]

*Get object of another domain*
```PowerView
Get-Domain -Domain moneycorp.local
```
![[Pasted image 20230606172106.png]]
```AD_module
Get-ADDomain -Identity moneycorp.local
```
*Get domain SID for the current domain*
```PowerView
Get-DomainSID
```
```AD_module
(Get-ADDomain).DomainSID
```
*Get domain policy for the current domain*
```PowerView
Get-DomainPolicyData
```
![[Pasted image 20230606172339.png]]
```POWERVIEW
(Get-DomainPolicyData).systemaccess
```
*Get domain controllers for the current domain*
```PowerView
Get-DomainController
```
![[Pasted image 20230606173931.png]]
```AD_module
Get_ADDomainController
```
*Get user information*
```PowerView
Get-DomainUser
```
```AD_MODULE
Get-ADUser -Filter * properties *
```
*Get domain computers information*
```PowerView
Get-DomainComputer
```
```AD_MODULE
Get-ADComputer -Filter * properties *
```
*Get domain groups information*
```PowerView
Get-NetGroup
```
```AD_MODULE
Get-ADGroup -Filter * properties *
```
*Find shares on hosts in current domain*
```PowerView
Invoke-ShareFinder –Verbose
```
*Find sensitive files on computers in the domain*
```PowerView
Invoke-FileFinder –Verbose
```
*Get all fileservers of the domain*
```PowerVieW
Get-NetFileServer
```

### **GPO ENUMERATION**
```PowerView
Get-DomainGPO
GET-NetGPO
```
```AD_module
Get-GPO -All 
```
### **ACL ENUMERATION**
 ```PowerView
Get-DomainObjectAcl
```
```powerview
Find-InterestingDomainAcl -ResolveGUIDs |  ?{$_.IdentityReferenceName -match "<USER>"}
```
 ```PowerView
Find-InterestingDomainAcl -ResolveGUIDs
```

### **TRUSTS ENUMERATION**
 ```PowerView
Get-DomainTrust
```
```AD_module
Get-ADTrust 
```

### **FORESTS ENUMERATION**
 ```PowerView
Get-Forest
Get-ForestTrust
```
```AD_module
Get-ADForest
```
### **USER HUNTING**
==Devi essere local admin==
```PowerView
Find-DomainuserLocation -Verbose
Find-DomainuserLocation -CheckAccess
```
```powershell
./Find-PSremotingLocalAdminAccess.ps1
Find-PSremotingLocalAdminAccess -verbose
```


### **BLOODHOUND**
Usa sharphound.ps1 o Invoke-BloodHound
```powershell
Invoke-Bloodhound -CollectionMethod All
```
Per evitare detection
```powershell
Invoke-Bloodhound -CollectionMethod All -ExcludeDC
```
