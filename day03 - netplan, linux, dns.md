# Change hostname, date-time, ip

## Change Hostname

```bash
cat /etc/hostname
hostnamectl set-hostname devops1
bash
cat /etc/hostname
```

## Change Timezone

```bash
timedatectl status
timedatectl set-timezone Asia/Kathmandu
```

## Change (Set) static ip

`/etc/netplan/*`

```yaml
# Old config
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: true

# New config
network:
  version: 2
  ethernets:
    ens33:
      addresses:
        - 192.168.22.99/24
      routes:
        - to: 0.0.0.0/0
          via: 192.168.22.2
      nameservers:
        addresses:
          - 1.1.1.1
```

```
sudo netplan apply
```

---

# DNS Configuration

Power DNS // Used by CPanel
named - BIND DNS // Better for inHouse

Bind utils - to better understand configs

Authorative DNS <---- Forwarding / Caching DNS

### Setup BIND

```
apt search bind9 bind9-utils
sudo apt -y install bind9 bind9-utils

sudo ss -tlnp | grep 53
# caching server -> where i should go to resolve
(("systemd-resolve",pid=675,fd=15))

# i am nameserver -> i will resolve request from others
users:(("named",pid=16972,fd=64))

systemctl status named

cd /etc/bind
cat named.conf
cat named.conf.options
cat named.conf.local
cat named.conf.default-zones
```

```
nslookup gmail.com
```

sudo nano named.conf.default-zones

```
######## named.conf.default-zones ########
------------ Add this
zone "devops.com" {
        type primary;
        file "/usr/share/dns/devops.local;
};
------------
// prime the server with knowledge of the root servers
<!-- zone "." {
        type hint;
        file "/usr/share/dns/root.hints";
}; --> comment this out

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
        type master;
        file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
        type master;
        file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
        type master;
        file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
        type master;
        file "/etc/bind/db.255";
};


```

cp /etc/bind/db.local /usr/share/dns/devops.local

Edit `/usr/share/dns/devops.local` this file after copying

```
;
; BIND data file for local loopback interface
;
$TTL 604800
@       IN   SOA     devops1.devops.com. root.devops.com. (
                          2      ; Serial
                     604800      ; Refresh
                      86400      ; Retry
                    2419200      ; Expire
                    604800 )     ; Negative Cache TTL
;
@       IN   NS     devops1.devops.com
@       IN   A      127.0.0.1
@       IN   AAAA   ::1
devops1 IN   A      192.168.22.99
www     IN   A      192.168.22.99

```

@ -> represents whole domain
A -> if no domain, respresnt this IP
AAAA -> for IPV6
devops1 -> static

named-checkzone, checkcong

restart named

options

uncomment forwarders

# DNS contd...

```
named-checkconf -c /etc/bind/named.conf.local
```

/etc/

---

`/etc/bind/devops.local`

```
;
; BIND data file for local loopback interface
;
$TTL 604800
@       IN   SOA     devops1.devops.com. root.devops1.devops.com. (
                          2      ; Serial
                     604800      ; Refresh
                      86400      ; Retry
                    2419200      ; Expire
                    604800 )     ; Negative Cache TTL
;
@       IN   NS     devops1.devops.com
devops1 IN   A      192.168.22.99
www     IN   A      192.168.22.99         // www to another
*       IN   A      192.168.22.101        // For wildcard
```

`'.' in the end represent full path: if '.' is not added, will add domain in the end`

---

`/etc/bind/named.conf.local`

```
zone "devops.com" {
        type master;
        file "/etc/bind/devops.local;
};
```

Reverse DNS

```
zone "22.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/192_168_22.local;
};
```

## From outside

nslookup
dig

### Self Study

- reverse dns
- pointer
- bind

```bash
#### /etc/bind/named.conf.options ####
options {
   directory "/var/cache/bind";

   // Allow queries from localhost and your local network
   allow-query { any; };
   allow-transfer { none; };

   // Enable recursion for trusted clients
   recursion yes;

   // Forward DNS queries to external DNS servers
   forwarders {
       8.8.8.8;
       8.8.4.4;
   };

   dnssec-validation no;
   version "not disclosed";

   // auth-nxdomain no;    # Disable authoritative NXDOMAIN responses
   // listen-on-v6 { none; };
   // listen-on { any; };
};

```

````bash
#### /etc/bind/named.conf.local ####
zone "dev.me" {
        type master;
        file "/etc/bind/db.dev.me";
};

```bash
#### /etc/bind/db.dev.me ####
$TTL    604800
@       IN      SOA     ns1.dev.me. admin.dev.me. (
                             2024101001         ; Serial
                             604800             ; Refresh
                             86400              ; Retry
                             2419200            ; Expire
                             604800 )           ; Negative Cache TTL
;
@       IN      NS      ns1.dev.me.
@       IN      A       192.168.22.97
ns1     IN      A       192.168.22.97
www     IN      A       192.168.22.97
wp      IN      A       192.168.22.97
*       IN      A       192.168.22.97
````
