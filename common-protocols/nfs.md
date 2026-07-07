# Basic Info 
- Several different versions 
	- NFSv2 
	- NFSv3 
	- NFSv4
- Ports 
	- 111
	- 249 
- More 
	- No mechanism for authentication or authorization 
	- Most common auth is UNIX UID/GID 

# Enumeration 


```
sudo nmap -sV -sC -p111,2049 --script nfs*

nuclei -u http://url -tags nfs 

showmount -e [ip-address]
```


```
mkdir target-nfs 
sudo mount -t nfs [ip-address]:/ ./target-nfs/ -o nolock 

```



```
ls -l mnt/nfs 

ls -n /mnt/nfs

cd .. 

sudo unmount ./target-NFS

```

# Exploitation 
