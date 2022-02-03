# High Availability Cluster with DRBD, Pacemaker and Corosync

## 1. Change hostnames
 hostnamectl set-hostname manish in primary node and node hostnamectl set-hostname mulchandani in secondary node
## 2. Allow permissions
 su to have permissions in both nodes
## 3. Add ips in hosts file
 vim /etc/hosts write ips and hostnames in both nodes
## 4. Copy hosts file in other node
 scp /etc/hosts mulchandani:/etc/ in primary node
## 5. Access other node
 ssh mulchandani in new tab in primary node
## 6. Flush ips
 iptables -F in both nodes
## 7. Check output type
 iptables -L in both nodes
## 8. Installing ntp
 rpm -qa | grep ntp and yum install -y ntp in both nodes
## 9. Configure ntp
 vim /etc/ntp.conf write server 127.127.1.0 in primary node and seconday node's ip in secondary node
## 10. Restart system ctl
 systemctl restart ntpd in both nodes
## 11.
 watch ntpq -p -n in both nodes
## 12. Disk info
 fdisk -l in both nodes
## 13. Create Partition
 fdisk /dev/sdb in both nodes
## 14. Create Physical Volume
 pvcreate /dev/sdb1 in both nodes
## 15. Create Volume Group
 vgcreate vgdrbd /dev/sbd1 in both nodes
## 16. Create Logical Volume
 lvcreate -n lvdrbd /dev/mapper/vgdrbd -L 4000M in both nodes
## 17. Configure system ctl
 vi /etc/sysctl.conf write text from video in both nodes
## 18. Copy systemctl.conf in other node
 scp /etc/sysctl.conf mulchandani:/etc/

## 1. Install drbd in both nodes
 rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
 rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
 yum install -y kmod-drbd84 drbd84-utils in both nodes
## 2. Drbf configuration
 vim /etc/drbd.conf then wrtie what is written in video in both nodes
## 3. To save drbd
 modprobe drbd on both machines in both nodes
## 4. Save on modprobe drbd in rc.localecho
 "modprobe drbd" >> /etc/rc.local in both nodes
## 5. Initilize drbd resources
 drbdadm create-md r0 in both nodes
## 6. To escape errors
 run failed cmds in both nodes
## 7. 
 drbdadm attach r0 in both nodes
## 8.
 groupadd haclient in both nodes
## 9.
 drbdadm syncer r0 in both nodes
## 10.
 drbdadm connect r0 in both nodes

## 1. Installing corosync and pacemaker
 yum install coresync pacemaker in both nodes
## 2. Uname
 uname -n in both nodes
## 3. Assigning  / Show iP
 ip a|grep "inet" in both nodes
## 4. Pinging
 ping -c1 manish in primary node and ping -c1 mulchandani in secondary node
## 5. Opening ports
 iptables -I INPUT -m state --state NEW -p udp -m multiport --dports 5404,5405 -j ACCEPT in both nodes
## 6. Opening ports
 iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 2224 -j ACCEPT in both nodes
## 7. Permissions
 iptables -I INPUT -p igmp -j ACCEPT in both nodes
## 8. Types
 iptables -I INPUT -m addrtype --dst-type MULTICAST -j ACCEPT in both nodes
## 9. Save iptables services
 service iptables save in both nodes
## 10. Assigning password must be same as root
 passwd hacluster then write password in both nodes
## 11. System ctl restart
 systemctl start pcsd in both nodes
## 12. Authenticating clusters
 pcs cluster auth manish mulchandani in primary node
## 13. Cluster setup
 pcs cluster setup --name cluster_web manish mulchandani in both nodes
## 14. Making it online
 pcs cluster start --all in both nodes
## 15. Showing Status
 pcs status cluster in both nodes

## 1. Install squid
 dnf install squid -y
## 2. Backup squid file
 sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.ori
## 3. Configure squid file
 vim /etc/squid/squid.conf
## 4. Allow firewall perms
 firewall-cmd --add-service=squid --permanent and firewall-cmd --reload
## 5. CentOS clinet configuration
 sudo vim /etc/profile.d/proxyserver.sh
## 6. Adding settings
 MY_PROXY_URL="szabist:3128" HTTP_PROXY=$MY_PROXY_URL HTTPS_PROXY=$MY_PROXY_URL FTP_PROXY=$MY_PROXY_URL http_proxy=$MY_PROXY_URL https_proxy=$MY_PROXY_URL ftp_proxy=$MY_PROXY_URL
## 8. Sourcing the file
 source /etc/profile.d/proxyserver.sh
