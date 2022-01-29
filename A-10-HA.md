<div id="top"></div>



<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
  </a>

  <h3 align="center">High Availability Cluster</h3>

  <p align="center">
    Built on Linux Centos 7 distribution! By Group A-10
    <br />
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project
Failover cluster or active-passive cluster are other terms for HA cluster. This form of cluster is one of the most extensively utilised in production environments because it ensures that services are available even if one of the cluster's nodes fails. If the server-side application fails for whatever reason, the pacemaker (cluster software) will restart the application on the active node. Failover is more than just restarting a programme; it's a set of related actions, such as mounting filesystems, configuring networks, and launching dependent apps.Multiple independent computers are grouped together to increase work availability
<p align="right">(<a href="#top">back to top</a>)</p>



### Built With

We have used these services/platform to make this High availability cluster.

* [Centos 7]
* [Putty]
* [DRBD]

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

We are using 2 machines:
* 20GB dedicated to each machine and one extra 10GB disk to server for shared storage.
* Hostname 		IP Address		Operating System	
* Server		192.168.64.135		Centos 7	
* Client		192.168.64.136		Centos 7		

### Prerequisites

Login all machines from root
* rpm -ivh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

* yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
  ```

<!-- USAGE EXAMPLES -->
## Usage
```sh
/etc/yum.repos.d/
yum info *drbd* | grep name
yum install drbd84-utils kmod-drbd84
rpm -qa | grep ntp
/etc/yum.repos.d/
vi /etc/ntp.conf
systemctl restart ntpd
systemctl enable ntpd
# ntpq -np
yum install ntp -y
```
For DNS Configuration

```sh
vi /etc/hosts

192.168.64.135   talha
192.168.6	4.136  atif

*scp to other vm*

ping talha
ping atif
```
For NTP Server
```sh
vi /etc/ntp.conf
systemctl restart ntpd
systemctl enable ntpd
```

For NTP Client
```sh
ntp cleint
vi /etc/ntp.conf
server 192.168.1.50
systemctl restart ntpd
systemctl enable ntpd
watch ntpq -p -n
```
Create Partition for DRDB
```sh
fdisk -l
fdisk /dev/sdb1
press p     that show drive is available
press m for help
n     for new partition
p for primary
partion number  must select 1
cylinder me enter
size    +4000M
system type:
press p partion
8e
w for save
partprobe partion restart
same for other vm
```
Create Physical Volume
```sh
pvcreate /dev/sdb1
create lvm partition
vgcreate vgdrbd /dev/sdb1
lvcreate -n lvdrbd /dev/mapper/vgdrbd -L +4000M
lvdisplay | more
vgdisplay | more
vi /etc/sysctl.conf
control source route verification
net.ipv4.conf.ens33.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.ens33.arp_announce = 2
sysctl -partion
scp to oth vm /etc/sysctl.conf
```
DRBD Setup
```sh
yum install -y drbd84 kmod-drbd84
vi /etc/drbd.conf
#delete the previous code of usr/share 
global {
    usage-count yes;
}
common {
   syncer {rate 10;}
}
resource r0 {
   protocol C;
   handlers {
	pri-on-incon-degr "echo o > /proc/sysrq-trigger ; halt -f";
	pri-lost-after-sb "echo o > /proc/sysrq-trigger ; halt -f";
	local-io-error "echo o > /proc/sysrq-trigger ; halt -f";
	outdate-peer "/usr/lib/heartbeat/drbd-peer-outdater -t 5";
   }
   startup {
   }
   disk {
	on-io-error detach;
   }
   net {
      after-sb-0pri disconnect;
      after-sb-1pri disconnect;
      after-sb-2pri disconnect;
      rr-conflict disconnect;
   }
   syncer {
      rate 10M;
      al-extents 257;
   }
   on talha {
      device   /dev/drbd0;
      disk     /dev/vgdrbd/lvdrbd;
      address  192.168.64.135:7788;
      meta-disk  internal;
   }
   on atif {
      device   /dev/drbd0;
      disk     /dev/vgdrbd/lvdrbd;
      address  192.168.64.136:7788;
      meta-disk  internal;
   }

}
modprobe drbd
```
```sh
echo "modprobe drbd" >> /etc/rc.local
cat /etc/rc.local
#to create resource
drbdadm create-md r0
chgrp haclient /sbin/drbdsetup
chmod 0-x /sbin/drbdsetup
chmod u+s /sbin/drbdsetup
chgrp haclient /sbin/drbdmeta
chmod 0-x /sbin/drbdmeta
chmod u+s /sbin/drbdmeta  
drbdadm up r0
lsblk
```
Firewall Connection
```sh
firewall-cmd
firewall-cmd --zone=internal --add-port=7788-7799/tcp --permanent
firewall-cmd --zone=internal --add-port=7788-7799/udp --permanent
firewall-cmd --reload
```
Start DRBD
```sh
systemctl start drbd
systemctl enable drbd
drbdadm primary r0 --force
drbdadm -- --overwrite-data-of-peer primary r0
drbdadm up all
watch cat /proc/drbd
```
```sh
user 1
mkfs.ext3 /dev/drbd0
mkdir /data/ (dono)
mount /dev/drbd0 /data/
cd /data/
ls
df -h
```
<p align="right">(<a href="#top">back to top</a>)</p>




<!-- CONTRIBUTING -->

## Contributing

1. Syed Muhammad Abu Talha
2. Atif Aziz Memon
3. Hafeez-Ur-Rehman Memon
<p align="right">(<a href="#top">back to top</a>)</p>
