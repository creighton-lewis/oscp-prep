```
Compromised host — where to move next?
│
├── SSH credentials available?
│   ├── Private keys found? → try on all known_hosts targets (§2)
│   ├── SSH agent running? → hijack socket (§1)
│   ├── Passwords in history/configs? → spray across hosts (§3)
│   └── authorized_keys writable on other hosts? → inject key (§2.3)
│
├── Network services discovered?
│   ├── Internal web apps? → tunnel + attack (§5.1)
│   ├── Databases (3306/5432/6379)? → check harvested creds (§3)
│   ├── SMB/NFS shares? → mount + search for creds/SUID (§6)
│   └── Kubernetes API (6443)? → load kubernetes-pentesting skill
│
├── Can reach other hosts?
│   ├── Direct SSH? → use keys/passwords
│   ├── Firewalled? → SSH tunnel or chisel (§5)
│   └── No tools? → /dev/tcp + bash (§5.2)
│
├── Root on current host?
│   ├── Read /etc/shadow → crack hashes → password reuse (§3)
│   ├── Dump /proc/*/environ → find service credentials (§3.2)
│   ├── Hijack sudo tokens → piggyback admin sessions (§7)
│   └── Modify systemd services → backdoor (§8)
│
├── D-Bus services available?
│   ├── Privileged services exposed? → method call abuse (§4)
│   └── polkit actions without auth? → privilege actions (§4.3)
│
└── No obvious path?
    ├── ARP scan + port sweep internal network (§5.3)
    ├── Passive credential sniffing (if cap_net_raw)
    ├── Wait for admin SSH → agent hijack (§1.3)
    └── Check for cloud metadata (169.254.169.254)
```
