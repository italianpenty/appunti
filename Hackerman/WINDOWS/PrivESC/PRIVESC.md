 *Tools*
- [enjoiz/Privesc: Windows batch script that finds misconfiguration issues which can lead to privilege escalation. (github.com)](https://github.com/enjoiz/Privesc)
- [PowerShellMafia/PowerSploit: PowerUP. (github.com)](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
- [AlessandroZ/BeRoot: Privilege Escalation Project - Windows / Linux / Mac (github.com)](https://github.com/AlessandroZ/BeRoot)
- [carlospolop/PEASS-ng: PEASS - Privilege Escalation Awesome Scripts SUITE (with colors) (github.com)](https://github.com/carlospolop/PEASS-ng)
### **FULL ENUMERATION**
```Powerup
Invoke-AllChecks
```
```Privesc
Invoke-PrivEsc
```
### **SERVICES**
*Unquoted paths and space in the name*
```Powerup
Get-ServiceUnquoted -Verbose
```
*Write in binary path*
```powerup
Get-ModifiableServiceFile -Verbose
```
*configuration current user can modify*
```powerup
Get-ModifiableService -Verbose
```
