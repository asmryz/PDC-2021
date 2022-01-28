HA (High Availability with DRBD & Heart Beat) DRBD (Distributed
Redundant Block Device) by this we can do the syncronization and
replication of the data between to devices. Heart Beat is a package.
(agar hum chahte hain k aik application 1 device ke down hone k bad
dusre device ke through run hojaye to uske is ke through chalate hain)
======================================================== Create two VMs
and intsall CentOS 7 on them.

Now Run All the commands one by One.
====================================

\#basic commands for the installation of repos.
===============================================

rpm -ivh
http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

yum install
https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm

ll /etc/yum.repos.d/

yum info *drbd* \| grep name

yum install drbd84-utils kmod-drbd84

rpm -qa \| grep ntp

/etc/yum.repos.d/

vi /etc/ntp.conf

systemctl restart ntpd

systemctl enable ntpd

ntpq -np
========

yum install ntp -y

========================== DNS config ===========================

vi /etc/hosts

192.168.64.135 talha 192.168.64.136 atif

*scp to other vm*

ping talha ping atif

========================= \#ntp sever ========================== vi
/etc/ntp.conf

systemctl restart ntpd

systemctl enable ntpd

========================= ntp cleint ==========================

ntp cleint

vi /etc/ntp.conf

server 192.168.1.50

systemctl restart ntpd

systemctl enable ntpd

watch ntpq -p -n

================================ \#Create partition for DRBD
================================

fdisk -l

fdisk /dev/sdb1

press p that show drive is available

press m for help

n for new partion

p for primary

partion number must select 1

cylinder me enter

size +4000M

system type:

press p partion

8e

w fo save

partprobe partion restart

same for other vm

==================================================== Create physical
volume ==================================================== pvcreate
/dev/sdb1

create lvm partion

=================================================

vgcreate vgdrbd /dev/sdb1

lvcreate -n lvdrbd /dev/mapper/vgdrbd -L +4000M

lvdisplay \| more

vgdisplay \| more

============================================== vi /etc/sysctl.conf

control source route verification

net.ipv4.conf.eth0.arp\_ignore = 1

net.ipv4.conf.all.arp\_announce = 2

net.ipv4.conf.eth0.arp\_announce = 2

sysctl -partion

======================================================== scp to oth vm
/etc/sysctl.conf

====================================================================================================

                 **********DRBD Setup*************

====================================================================================================
yum install -y drbd84 kmod-drbd84

===================================

vim /etc/drbd.conf

\#delete the previous code of usr/share

global { usage-count yes; }

common { syncer {rate 10;} }

resource r0 { protocol C; handlers { pri-on-incon-degr "echo o \>
/proc/sysrq-trigger ; halt -f"; pri-lost-after-sb "echo o \>
/proc/sysrq-trigger ; halt -f"; local-io-error "echo o \>
/proc/sysrq-trigger ; halt -f"; outdate-peer
"/usr/lib/heartbeat/drbd-peer-outdater -t 5"; }

startup { }

disk { on-io-error detach; }

net { after-sb-0pri disconnect; after-sb-1pri disconnect; after-sb-2pri
disconnect; rr-conflict disconnect; }

syncer { rate 10M; al-extents 257; }

on talha { device /dev/drbd0; disk /dev/vgdrbd/lvdrbd; address
192.168.64.135:7788; meta-disk internal; }

on atif { device /dev/drbd0; disk /dev/vgdrbd/lvdrbd; address
192.168.64.136:7788; meta-disk internal; }

}

========================= modprobe drbd

==========================

echo "modprobe drbd" \>\> /etc/rc.local

cat /etc/rc.local

=====================================

\#to create resource

drbdadm create-md r0

================================

chgrp haclient /sbin/drbdsetup chmod 0-x /sbin/drbdsetup chmod u+s
/sbin/drbdsetup

chgrp haclient /sbin/drbdmeta chmod 0-x /sbin/drbdmeta chmod u+s
/sbin/drbdmeta

=================================

drbdadm up r0 lsblk

firewall-cmd

firewall-cmd --zone=internal --add-port=7788-7799/tcp --permanent

firewall-cmd --zone=internal --add-port=7788-7799/udp --permanent

firewall-cmd --reload

systemctl start drbd systemctl enable drbd

drbdadm primary r0 --force

drbdadm -- --overwrite-data-of-peer primary r0

drbdadm up all

watch cat /proc/drbd

================ user 1

mkfs.ext3 /dev/drbd0

mkdir /data/ (dono)

mount /dev/drbd0 /data/ cd /data/ ls df -h
