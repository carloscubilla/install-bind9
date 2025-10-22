# install-bind9

#### 1. Install bind9
```powershell
 dnf install bind bind-utils -y
 systemctl enable --now named
 systemctl status named
```

#### 2. Config named.conf
```powershell
vim /etc/named.conf
```

#### 3. Contenido de named.conf
```powershell
options {
    listen-on port 53 { any; };
    listen-on-v6 port 53 { none; };
    directory       "/var/named";
    dump-file       "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    allow-query     { any; };

    recursion yes;
    forwarders { 8.8.8.8; 1.1.1.1; };

    dnssec-validation yes;
    managed-keys-directory "/var/named/dynamic";
    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";
    include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
    channel default_debug {
        file "data/named.run";
        severity dynamic;
    };
};

zone "." IN {
    type hint;
    file "named.ca";
};

# Zonas del dominio real
zone "ocp.netlogic.com.py" IN {
    type master;
    file "/var/named/ocp.netlogic.com.py.zone";
    allow-update { none; };
};

zone "220.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/220.168.192.rev";
    allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```



#### 4. Crear zona directa
```powershell
vim /var/named/ocp.netlogic.com.py.zone
```


#### 5. Contenido de zona directa
```powershell
$TTL 1D
@   IN  SOA  dns.ocp.netlogic.com.py. root.ocp.netlogic.com.py. (
        2025102201 ; Serial
        1H         ; Refresh
        15M        ; Retry
        1W         ; Expire
        1D )       ; Minimum TTL

    IN  NS  dns.ocp.netlogic.com.py.

dns         IN  A  192.168.220.5
api         IN  A  192.168.220.10
api-int     IN  A  192.168.220.10
*.apps      IN  A  192.168.220.10
bootstrap   IN  A  192.168.220.11
master0     IN  A  192.168.220.12
master1     IN  A  192.168.220.13
master2     IN  A  192.168.220.14
worker0     IN  A  192.168.220.15
worker1     IN  A  192.168.220.16
```


#### 6. Crear zona inversa
```powershell
vim /var/named/220.168.192.rev
```



#### 7. Contenido de zona inversa
```powershell
$TTL 1D
@   IN  SOA  dns.ocp.netlogic.com.py. root.ocp.netlogic.com.py. (
        2025102201 ; Serial
        1H
        15M
        1W
        1D )

    IN  NS  dns.ocp.netlogic.com.py.

5   IN  PTR  dns.ocp.netlogic.com.py.
10  IN  PTR  api.ocp.netlogic.com.py.
11  IN  PTR  bootstrap.ocp.netlogic.com.py.
12  IN  PTR  master0.ocp.netlogic.com.py.
13  IN  PTR  master1.ocp.netlogic.com.py.
14  IN  PTR  master2.ocp.netlogic.com.py.
15  IN  PTR  worker0.ocp.netlogic.com.py.
16  IN  PTR  worker1.ocp.netlogic.com.py.
```


#### 8. Aplicar permisos
```powershell
chown named:named /var/named/220.168.192.rev
chmod 640 /var/named/220.168.192.rev
```



#### 9. Validar configuraciónos
```powershell
named-checkconf
named-checkzone ocp.netlogic.com.py /var/named/ocp.netlogic.com.py.zone
named-checkzone 220.168.192.in-addr.arpa /var/named/220.168.192.rev
```


#### Deberías ver:

 zone ocp.netlogic.com.py/IN: loaded serial 2025102201
 OK
 zone 220.168.192.in-addr.arpa/IN: loaded serial 2025102201
 OK
