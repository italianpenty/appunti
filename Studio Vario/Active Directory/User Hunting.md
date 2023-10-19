Invoke-UserHunte
1) query the members of the target group (“Domain Admins” by default).  
2) query the domain for all machines using Get-NetComputers. 
3) perform a Get-NetSessions and Get-NetLoggedOn against every host in the list and look for the users previously queried. 
While this technique provides the most coverage, it has the possibility of being slow and noisy-ish depending on the network (though we rarely get caught with it).
