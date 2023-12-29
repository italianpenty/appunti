![[Pasted image 20231220160920.png]]

Cannot create type. Only core types are supported in this language mode.

To check if you're running in constrained language mode on local
```
reg query 
HKLM\Software\Policies\Microsoft\Windows\SRPV2
```
on remote 
```
reg query 
HKLM\Software\Policies\Microsoft\Windows\SRPV2 -ComputerName <MACHINE NAME>
```
![[Pasted image 20231220161133.png]]
```
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections -ComputerName <MACHINE NAME>
```
![[Pasted image 20231220161307.png]]
It say that we can run program inside C:\\Program Files folder, so download the script inside it and run it