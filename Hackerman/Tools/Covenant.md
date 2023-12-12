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