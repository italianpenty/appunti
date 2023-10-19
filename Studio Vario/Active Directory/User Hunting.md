Invoke-UserHunter
1) query the members of the target group (“Domain Admins” by default).  
2) query the domain for all machines using Get-NetComputers. 
3) perform a Get-NetSessions and Get-NetLoggedOn against every host in the list and look for the users previously queried. 
While this technique provides the most coverage, it has the possibility of being slow and noisy-ish depending on the network (though we rarely get caught with it).

*Derivate Local Admin*
![[Pasted image 20231019113053.png]]
1. We Phish Sally and escalate our privileges on her system. We can use Mimikatz to extract her credentials, giving us the rights of the “Workstation Admins” group
2. Invoke-StealthUserHunter -ShowAll finds a Domain Admin logged in on WorkstationA
3. We do a Get-NetLocalGroup WorkstationA and see that there is a group “Network Ops” on it
4. We use Get-NetGroup “Network Ops” to enumerate users we want to target next. We see “Fred” is a member of this domain group, and match this up with our previous user hunting data to see that he’s logged in to WorkstationB
5. We run a Get-NetLocalGroup WorkstationB to find that there is a group “Workstation Admins”
6. We use our credentials to gain access to WorkstationB with WMI or PSExec and the agent of your choice
7. We use Invoke-Mimikatz to dump the credentials from WorkstationB and find Fred’s credentials
8. We use Fred’s credentials to gain access to WorkstationA
9. We use Invoke-Mimikatz to dump credentials and retrieve the Domain Admin creds!
