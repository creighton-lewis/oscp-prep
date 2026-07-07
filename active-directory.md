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

## Lateral Movement 
## Privilege Escalation 
## Exploitation (from Linux)
## Exploitation (from Windows )