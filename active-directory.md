# Active Directory 
## External Enumeration 
## Internal Enumeration 
### AD Module 
```
Get-Module 
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
Get-DomainController
```


```
Get DomainUser 
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
```

## Lateral Movement 
## Privilege Escalation 
## Exploitation (from Linux)
## Exploitation (from Windows )