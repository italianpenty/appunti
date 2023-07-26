Per capire quale parte del codice bisogna cambiare bisogna "spezzare" il codice per poi capire quale parte è necessario cambiare.

Find-AVSignature è uno script di powersploit
```link
https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/AntivirusBypass/Find-AVSignature.ps1
```
*-Interval*    grandezza segmenti (BYTES)
*-Path*         input file
*-OutPath*   output folder
*-Verbose -Force*    più logs e forzi la creazione della cartella specificata
```powershell
Find-AVSignature -StartByte 0 -EndByte max -Interval 1000 -Path C:\Tools\met.exe -OutPath C:\Tools\avtest1 -Verbose -Force
```
![[Pasted image 20230505164229.png]]
Ora testa i vari pezzi di codice con un antivirus per vedere quali parti cambiare
```bash
clamscan <PATH>
```
![[Pasted image 20230505164358.png]]
Ora continua a riderre i bite fnche non trovi quelli che flaggano l'av

==CONTINUARE DA PEN-300 QUANDO AVRAI UN PC SBLOCCATO :)==



