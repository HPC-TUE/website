# Installation

![DNS Logo](dns.png){ align=right heigth="100" }

## Requirements

A succesfull (Non HA) installation (hpc-head01) of TrinityX using the ansible-playbook.

A second server (hpc-head02) with a basic Rocky 8 install 

## Server Configuration

#### head01:

```shell
cat > /trinity/local/luna/daemon/templates/templ_dns_zones_conf.cfg << EOF
{% autoescape None %}

{% for ZONE in ZONES %}
zone "{{ ZONE }}" IN {
    type master;
    file "/var/named/{{ ZONE }}.luna.zone";
    allow-update { none; };
    allow-transfer { 10.150.255.253; };
    also-notify { 10.150.255.253; };
};
{% endfor %}

{% endautoescape %}
EOF
```

```shell
vi /trinity/local/luna/daemon/templates/templ_dhcpd.cfg

   option domain-name-servers 10.150.255.254, 10.150.255.253;
```
#### head02:

Make sure the DHPC on the external interface (eno1) does not overwrite DNS entries in resolv.conf
```shell
sed -i 's/main]/main]\ndns=none/' /etc/NetworkManager/NetworkManager.conf
```

```shell
dnf -y install bind

cat > /etc/named.conf << EOF
options {
        listen-on port 53 { 127.0.0.1;10.150.255.253; };
        listen-on-v6 port 53 { ::1; };
        statistics-file "/var/lib/named/data/named_stats.txt";
        memstatistics-file "/var/lib/named/data/named_mem_stats.txt";
        secroots-file   "/var/lib/named/data/named.secroots";
        recursing-file  "/var/lib/named/data/named.recursing";
        dump-file       "/var/lib/named/data/cache_dump.db";
        directory       "/var/named";
        allow-query { 127.0.0.0/8;10.150.0.0/16; };
        allow-notify { 10.150.255.254;};
        recursion yes;
        forwarders { 131.155.2.3;131.155.3.3; };
        dnssec-enable no;
        dnssec-validation no;
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
        include "/etc/crypto-policies/back-ends/bind.config";
};
logging {
        channel default_debug { file "data/named.run"; severity dynamic; };
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
include "/etc/named.luna.zones";
EOF
```
```shell
cat > /etc/named.luna.zones << EOF
zone "cluster" IN {
    type slave;
    file "/var/named/cluster.luna.zone";
    masters {10.150.255.254; };
};
zone "150.10.in-addr.arpa" IN {
    type slave;
    file "/var/named/150.10.in-addr.arpa.luna.zone";
    masters {10.150.255.254; };
};
zone "ipmi" IN {
    type slave;
    file "/var/named/ipmi.luna.zone";
    masters {10.150.255.254; };
};
zone "16.172.in-addr.arpa" IN {
    type slave;
    file "/var/named/16.172.in-addr.arpa.luna.zone";
    masters {10.150.255.254; };
};
zone "ib" IN {
    type slave;
    file "/var/named/ib.luna.zone";
    masters {10.150.255.254; };
};
zone "149.10.in-addr.arpa" IN {
    type slave;
    file "/var/named/149.10.in-addr.arpa.luna.zone";
    masters {10.150.255.254; };
};
EOF
```
```shell
cat > /etc/resolv.conf << EOF
search cluster ipmi
nameserver 10.150.255.253
nameserver 131.155.2.3
nameserver 131.155.3.3
EOF
```

```shell
systemctl enable named --now
```

## Umbrella Zone

### hpc-head01
```shell
echo 'include "/etc/named.umbrella.zones";' >> /etc/named.conf

cat > /etc/named.umbrella.zones << EOF
zone "umbrella" IN {
    type master;
    file "/var/named/umbrella.zone";
    allow-update { none; };
    allow-transfer { 10.150.255.253; };
    also-notify { 10.150.255.253; };
};
EOF

cat > /var/named/umbrella.zone << EOF
\$TTL 3600
@ IN SOA                controller.umbrella. root.controller.umbrella. ( ; domain email
                        1723024849 ; serial number
                        86400        ; refresh
                        14400        ; retry
                        3628800      ; expire
                        3600 )       ; min TTL

                        IN NS controller.umbrella.

controller                      IN A 10.150.255.254
vast-storage                    IN A 10.150.250.1
vast-storage                    IN A 10.150.250.2
vast-storage                    IN A 10.150.250.3
vast-storage                    IN A 10.150.250.4
vast-storage                    IN A 10.150.250.5
vast-storage                    IN A 10.150.250.6
vast-storage                    IN A 10.150.250.7
vast-storage                    IN A 10.150.250.8
vast-storage                    IN A 10.150.250.9
vast-storage                    IN A 10.150.250.10
vast-storage                    IN A 10.150.250.11
vast-storage                    IN A 10.150.250.12
vast-storage                    IN A 10.150.250.13
vast-storage                    IN A 10.150.250.14
vast-storage                    IN A 10.150.250.15
vast-storage                    IN A 10.150.250.16

EOF

systemctl reload named.service
```

### hpc-head02

```shell
echo 'include "/etc/named.umbrella.zones";' >> /etc/named.conf

cat > /etc/named.umbrella.zones << EOF
zone "umbrella" IN {
    type slave;
    file "/var/named/umbrella.zone";
    masters {10.150.255.254; };
};
EOF

systemctl reload named.service
```
