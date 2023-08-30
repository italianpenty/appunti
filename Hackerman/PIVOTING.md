### **SSH**
### **SSHUTTLE**
You can use sshuttle to tunnel all traffic to a remote SSH server. Note that sshuttle only forwards DNS requests and TCP traffic to the remote server. Other protocols, like UDP, are not supported.

REQUISITI:
• For a start, sshuttle only works on **Linux targets**.
• It also requires **access** to the compromised server **via** **SSH**,
• and Python also needs to be installed on the server.

The base command for connecting to a server with sshuttle is as follows:

`sshuttle -r username@address subnet` 

For example, in our fictional 172.16.0.x network with a compromised server at 172.16.0.5, the command may look something like this:

`sshuttle -r user@172.16.0.5 172.16.0.0/24`

We would then be asked for the user's password, and the proxy would be established. The tool will then just sit passively in the background and forward relevant traffic into the target network.

Rather than specifying subnets, we could also use the `-N` option which attempts to determine them automatically based on the compromised server's own routing table:

`sshuttle -r username@address -N` (Bear in mind that this may not always be successful though!)

As with the previous tools, these commands could also be backgrounded by appending the ampersand (`&`) symbol to the end.

If this has worked, you should see the following line:

`c : Connected to server.`

How to use sshuttle with key files for authentication

#It's not directly mentioned in the documentation on how to do this, so here you go. This command will tunnel everything including DNS:

`sshuttle --dns -vr user@yourserver.com 0/0 --ssh-cmd 'ssh -i /your/key/path.pem'`

`sshuttle --dns -vr root@10.200.105.200 0/0 --ssh-cmd 'ssh -i /home/kali/.ssh/whealth_priv_key' &`

`sshuttle -r user@172.16.0.5 --ssh-cmd "ssh -i private_key" 172.16.0.0/24`

![file://C:/Users/STRAOL~1/AppData/Local/Temp/.Y3VCA2/1.png](file://C:/Users/STRAOL~1/AppData/Local/Temp/.Y3VCA2/1.png)

**Please Note:** When using sshuttle, you may encounter an error that looks like this:

`client: Connected.`

`client_loop: send disconnect: Broken pipe`

`client: fatal: server died with error code 255`

This can occur when the compromised machine you're connecting to is part of the subnet you're attempting to gain access to. For instance, if we were connecting to 172.16.0.5 and trying to forward 172.16.0.0/24, then we would be including the compromised server inside the newly forwarded subnet, thus disrupting the connection and causing the tool to die.

To get around this, we tell sshuttle to exclude the compromised server from the subnet range using the `-x` switch.

To use our earlier example:

`sshuttle -r user@172.16.0.5 172.16.0.0/24 -x 172.16.0.5`

This will allow sshuttle to create a connection without disrupting itself.

TEST:

`sudo tcptraceroute <ip_of_subnet_unrichable_before>`

PING NON FUNZIONA