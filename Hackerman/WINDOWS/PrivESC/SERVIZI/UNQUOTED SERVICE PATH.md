Trova un servizio con il service path unquoted e che starti come system
![[Pasted image 20230503145353.png]]
Nota che puoi scrivere nella directory program files/unquoted Path Sevice/
Copia la reverse shell che hai creato in questa directory e rinominala Common.exe
Rilancia il servizio
```POWERSHELL
net start unquotedsvc
```
