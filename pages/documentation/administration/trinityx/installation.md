# Installation Prod

| Nodes                     |      IPMI      |   Internal IP   | External IP (DHCP) |
|:--------------------------|:--------------:|:---------------:|:------------------:|
| hpc-head01 ***TrinityX*** | 172.16.108.55  | 10.150.255.254  |   131.155.7.102    |
| hpc-head02                | 172.16.108.56  | 10.150.255.253  |   131.155.7.103    |
| tue-storage001            | 172.16.108.151 |  10.150.254.1   |        N/A         |
| VAST                      |      N/A       | 10.150.250.0/24 |        N/A         |

## Requirements

- Node with at least 2 NIC interfaces (public & internal network)
- [Rocky Linux](https://rockylinux.org){:target=_blank} 8.x
- iDrac access to hpc-head01 and hpc-head02

??? example "Configuration"

    **Localization**
    
    - Keyboard: English (US)
    - Language Support: English (United States)
    - Time & Date: Europe/Amsterdam
    
    
    **Software**
    
    - Software Selection: Minimal Install (Standard)
    
    **System**
    
    - Installation Destination: Local Standard Disks (sda), Storage Configuration: Automatic
    - Network & Hostname: Configure Network, Hostname: hpc-head0X.icts.tue.nl
    
    **User Settings**
    
    - User Creation: ********, Make this user administrator

## Server Configuration

As root@hpc-head0X

**TIP**: use tmux to mitigate connection loss during installation

```shell
dnf -y install tmux
```

### Network Configuration

Configure internal network interface

=== "hpc-head01"

    ```shell
    export IFACE=ens3f0np0   

    #nmcli connection add type ethernet con-name $IFACE
    nmcli con mod $IFACE ipv4.address 10.150.255.254/16
    nmcli con mod $IFACE ipv4.method manual
    nmcli con mod $IFACE connection.autoconnect true
    nmcli con up $IFACE
    ```

    ```shell
    ip address | grep -B4 172
    nmcli connection show  # Select correct device UUID.
    nmcli connection modify  UUID  ipv.never-default true
    ```

=== "hpc-head02"

    ```shell
    export IFACE=ens3f0np0    

    #nmcli connection add type ethernet con-name $IFACE
    nmcli con mod $IFACE ipv4.address 10.150.255.253/16
    nmcli con mod $IFACE ipv4.method manual
    nmcli con mod $IFACE connection.autoconnect true
    nmcli con up $IFACE

    firewall-cmd --zone=trusted --change-interface=$IFACE --permanent
    firewall-cmd --reload

    dnf -y install nfs-utils
    ```

    ```shell
    ip address | grep -B4 172
    nmcli connection show  # Select correct device UUID.
    nmcli connection modify  UUID  ipv.never-default true
    ```

!!! Note

    Make sure to use the correct name of your internal NIC.

### Update the OS

```shell
dnf -y update
```

### Reboot System

```shell
reboot
```

## TrinityX Installation

### Prepare environment (hpc-head01 and hpc-head02)

As `root@hpc-head0X`:

```shell
dnf -y install git
```

Make sure the DHPC on the external interface (eno1) does not overwrite DNS entries in resolv.conf

```shell
sed -i 's/main]/main]\ndns=none/' /etc/NetworkManager/NetworkManager.conf
```

Clone the TrinityX github repo

```shell
git clone https://github.com/clustervision/trinityX.git /root/trinityX
cd /root/trinityX
bash prepare.sh
```

### Configure environment (hpc-head01 only)

```shell
cp site/group_vars/all.yml.example site/group_vars/all.yml
```

Review and edit the contents of the `all.yml` file accordingly, notable settings:

| Setting                      | Value                         | Description                                                 |
|------------------------------|-------------------------------|-------------------------------------------------------------|
| administrator_email          | `hpc-umbrella@tue.nl`         | Email address of the administrator                          |
| project_id                   | `umbrella`                    | Project ID                                                  |
| ha                           | `false` (default)             | High Availability; _MUST remain `false` at time of writing_ |
| trix_ctrl1_ip                | `10.150.255.254 `             | IP controller node in Cluster Network                       |
| trix_ctrl1_bmcip             | `172.16.108.??`               | IP controller node in BMC Network                           |
| trix_ctrl1_hostname          | `hpc-head01`                  | Hostname                                                    |
| trix_cluster_net.            | `10.150.0.0`                  | Cluster (Private) Network                                   | 
| trix_cluster_netprefix.      | `16`                          | CIDR of Cluster Network                                     |
| trix_cluster_dhcp_start      | `10.150.128.0`                | Start of DHCP range for nodes                               |
| trix_cluster_dhcp_end        | `10.150.135.255`              | End of DHCP range for nodes                                 |
| trix_external_fqdn           | `umbrella-cluster.hpc.tue.nl` | FQDN of the external interface of the cluster               |
| trix_dns_forwarders          | `[131.155.2.3, 131.155.3.3]`  | List of DNS forwarders to use for the cluster.              |
| firewalld_public_interfaces  | `[eno1]`                      | List of public interfaces to use for the cluster.           |
| firewalld_trusted_interfaces | `[ens3f0np0]`                 | List of trusted interfaces to use for the cluster.          |
| el8_openhpc_repositories     | `OpenHPC/2/update.2.6.2/EL_8` | Activate latest update to OpenHPC 2.6                       |

```shell
cp site/hosts.example site/hosts
```

Review and edit the contents of the `hosts` file accordingly.

```ini
[controllers]
hpc-head01 ansible_host=10.150.255.254 ansible_connection=local
#hpc-head02 ansible_host=10.150.255.253
```

### Installation (hpc-head01 only)

Run the ansible-playbook

```shell
cd /root/trinityX/site
ansible-playbook controller.yml
```

### Configuration

#### Set cluster settings

```shell
luna cluster change -n umbrella -c hpc-umbrella@tue.nl
```

#### Configure the BMC(ipmi) network

```shell
luna network change -N 172.16.108.0/23 ipmi
```

#### Fix uchiwa logrotate-script owner

```shell
chown root /etc/logrotate.d/uchiwa 
```

### Create a demo user (or not)

```shell
obol user add demo -p demo
obol group addusers admins demo
```

Now the following Pages should available:

http://umbrella-cluster.hpc.tue.nl:8080  TrinityX (use demo:demo)

http://umbrella-cluster.hpc.tue.nl:3000  Grafana

http://umbrella-cluster.hpc.tue.nl:3001  Sensu

#### Create the basic OSimage (compute)

```shell
ansible-playbook compute-redhat.yml
luna osimage pack compute
```

#### Add [pre.txt](pre.txt){:target=_blank}, [part.txt](part.txt){:target=_blank} and [post.txt](post.txt){:target=_blank} to the part/post of the compute image

```shell
luna group change -qpre pre.txt compute
luna group change -qpart part.txt compute
luna group change -qpost post.txt compute
```

Authentication via AD

```shell
cat >> /etc/sssd/sssd.conf << EOF

auth_provider = krb5
krb5_server = campus.tue.nl
krb5_kpasswd = campus.tue.nl
krb5_realm = CAMPUS.TUE.NL
cache_credentials = True
EOF
systemctl restart sssd
```

#### BMC network

```shell
export IFACE=eno2   

nmcli connection add type ethernet con-name $IFACE
nmcli con mod $IFACE ipv4.method auto
nmcli con mod $IFACE connection.autoconnect true
nmcli con up $IFACE
```

*[NIC]: Network Interface Controller
