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
