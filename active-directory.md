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
Get-DomainOU
```

```
FindINterestingDomainAcl

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
```
```
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

## Lateral Movement 
## Privilege Escalation 
## Exploitation (from Linux)
## Exploitation (from Windows )