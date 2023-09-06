### **ENUMERATION**
```bash
dig axfr @<IP> <NOME DOMINIO>
```
![[Pasted image 20230508172556.png]]

### **DNS HIJACKING**
Se tramite un lfi riesci a dumpare la secret key del dns presente in
```PATH
/etc/bind/named.conf
```
![[Pasted image 20230508173833.png]]
Puoi provare a sostituire un subdomain per reinderizzare il tuo traffico su di te(Ad esempio con le mail di reset delle password {snoopy.htb})
___
Esportare la chiave nel formato
```BASH
export HMAC=<ALGORITMO>:<NOME CHIAVE>:<CHIAVE>
```
E poi usa il comando 
```bash
nsupdate -y $HMAC
```
per modificare il dns.
___
Maggiori informazioni a [DNS Updates with nsupdate - Serverless Industries](https://serverless.industries/2020/09/27/dns-nsupdate-howto.en.html)
