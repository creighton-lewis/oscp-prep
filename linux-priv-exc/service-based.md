# Linux Capabilities 
- Capabilities granted to specific programs that may occassionally contain excessive privileges

**Enumeration**
  
```
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;
```
Linux Capabilities that can be abused: 
- cap_dac_override
- cap_sys_admin
- cap_audit_control
- cap_audit_write
- cap_chown

# Cron Jobs 
## Enumeration 

```
crontab -l
crontab -e
-find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```
# Containers 
## Docker 
Practice 
[Vulnerable Container](https://github.com/haltacademy/Linux-Privilege-Escalation-Docker-Lab)
```
id #helps us find out if we are part of docker group
docker image ls #answers what docker images we have access to 
```
## Polkit

```
psexec -u <user> <command>  

psexec -u root id

```
## Kubernates 
## Logrotate 
## Misc 
