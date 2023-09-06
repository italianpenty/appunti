_AMSI Trigger_ 
[RythmStick/AMSITrigger: The Hunt for Malicious Strings (github.com)](https://github.com/RythmStick/AMSITrigger)

```
AmsiTrigger_x64.exe -i C:\AD\Tools\Invoke-PowerShellTcp_Detected.ps1
```

_INVOKE OBFUSCATION_
[danielbohannon/Invoke-Obfuscation: PowerShell Obfuscator (github.com)](https://github.com/danielbohannon/Invoke-Obfuscation)

---

**PER BYPASSARE L'AMSI**

1. Scan usando AMSITrigger
2. Modifica la porzione di codice cioccata
3. Rescan con AMSITrigger
4. Ripeti step 2 e 3 finchè non ottieni ==AMSI_RESULT_NOT_DETECTED== o ==BLANK==