1) PortScan
```bash
rustscan -a 10.13.37.10 --ulimit 5000 -- -sV -sC -Pn
```
![[Pasted image 20230913115945.png]]
2) Digging the domain
```bash
dig axfr @10.13.37.10-x 10.13.37.10
```
3) 