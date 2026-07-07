
 # Usage 
 
# Enumeration  

```
nc -vn X.X.X.X 22
```
```
ssh htb-student@10.10.10.10
```
```
sudo nmap -sV --script ssh* 
```

```
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit

./ssh-audit.py 10.129.14.132
```

```

msfconsole 
search ssh_enumusers 
search ssh_login 

```



```
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null

```


```
ssh-keygen -yf ~/.ssh/id_ed25519 # this checks to see if the ssh key id_ed255519 is actually encrypted
```

```
ssh2john.py SSH.private > ssh.hash
john --wordlist=rockyou.txt ssh.hash
```

# Exploitation 

Brute forcing 

```
git clone https://github.com/urbanadventurer/username-anarchy
cd username-anarchy firstname lastname 
```

```
hydra -l user -P /usr/share/wordlists/seclists/

```

![[Linux Port Forwarding Techniques#Ssshuttle]]
# Post Exploitation 
*Port Forwarding*

```
ssh -R remoteport:localhost:localPort user@sshServer
```


```
ssh-keygen -f key 

```
- Creates two files: key (used to log into remote machine)
- key.pub (copied to remote machine and added to /root/.ssh/authorized_keys) 
- Should be able to log in as that user by using private key 
```
ssh root@10.10.10.10 -i key 

```
