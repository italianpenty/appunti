### **NETWORK**
Per uno scanning manuale delle porte puoi usare il seguente one-liner
![[Pasted image 20230505112237.png]]
Oppure usare il modulo Invoke-Port Scan di PowerSploit Framework
```LINK
https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/Invoke-Portscan.ps1
```
### **SID**
```powershell
whoami /all | Select-String -Pattern "<USER>" -Context 2,0
```
### **CESTINO**
```powershell
cd 'C:\$Recycle.bin\<SID>'
```


