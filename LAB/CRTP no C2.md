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
