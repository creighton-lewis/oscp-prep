
# Proxifier 

# Plink.exe

```
powershell -c curl https://the.earth.li/~sgtatham/putty/latest/w64/plink.exe -Outfile plink.exe

```

```
plink.exe

```

# Netsh

```
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```

```
netsh.exe interface portproxy show v4tov4
```
# SocksOverRDPx64
