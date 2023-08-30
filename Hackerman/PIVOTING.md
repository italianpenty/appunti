### **SSH**
### **SSHUTTLE**
You can use sshuttle to tunnel all traffic to a remote SSH server. Note that sshuttle only forwards DNS requests and TCP traffic to the remote server. Other protocols, like UDP, are not supported.

REQUISITI:
• For a start, sshuttle only works on Linux targets.
• It also requires access to the compromised server via SSH.
• and Python also needs to be installed on the server.

The base command for connecting to a server with sshuttle is as follows:
```bash
sshuttle -r username@address subnet
```
For example, in our fictional 172.16.0.x network with a compromised server at 172.16.0.5, the command may look something like this:
```bash
shuttle -r user@172.16.0.5 172.16.0.0/24
```
Rather than specifying subnets, we could also use the `-N` option which attempts to determine them automatically based on the compromised server's own routing table:
```bash
sshuttle -r username@address -N
```
(Bear in mind that this may not always be successful though!)
If this has worked, you should see the following line:

`c : Connected to server.`

This command will tunnel everything including DNS:
```bash
sshuttle --dns -vr user@yourserver.com 0/0 --ssh-cmd 'ssh -i /your/key/path.pem'
```

```bash
sshuttle -r user@172.16.0.5 --ssh-cmd "ssh -i private_key" 172.16.0.0/24
```
![file://C:/Users/STRAOL~1/AppData/Local/Temp/.Y3VCA2/1.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.Y3VCA2/1.png)

sshuttle to exclude the compromised server from the subnet range using the `-x` switch.

To use our earlier example:
```bash
sshuttle -r user@172.16.0.5 172.16.0.0/24 -x 172.16.0.5
```
This will allow sshuttle to create a connection without disrupting itself.

TEST:

`sudo tcptraceroute <ip_of_subnet_unrichable_before>`

PING NON FUNZIONA