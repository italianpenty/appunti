- https://github.com/ropnop/windapsearch
### **EXTERNAL ENUMERATION**
[Releases · ropnop/go-windapsearch (github.com)](https://github.com/ropnop/go-windapsearch/releases)

### **LDAPSEARCH**
ottieni un ticket di kerberos
modifica /etc/krb5.conf e usa il comando 
```BASH
kinit <nome utente>
```
per verificare
```BASH
klist
```
![[Pasted image 20230621113454.png]]
Ora puoi usare ldapsearch
```BASH
ldapsearch -H ldap://<MACCHINA< -D '<DOMINIO>\<USER>' -w '<USER>' -Y GSSAPI - b "cn=users,dc=scrm,dc=local" | grep -i "objectSid::" | cut -d ":" -f3
```
### **DOMAIN DUMP**
*CON ACCESSO AL SISTEMA*
```BASH
ldapdomaindump $IP -u '<DOMAIN NAME>\<USER>' -p <PASS> --no-json --no-grep
```
![[Pasted image 20230510114519.png]]
*SILVER TICKET*
Con il silver ticket puoi dumpare l'ldap
```bash
crackmapexec ldap <NOME DOMINIO> -u <USER> -k --kdcHost <DC> --users <IP>
```
