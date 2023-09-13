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
3) 