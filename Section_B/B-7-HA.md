
# High Availability Cluster with DRBD, Pacemaker/Corosync

  

We are going to use DRBD (Distributed Redundant Block Device), Pacemaker/Corosync for Synchronization and High Availability Cluster will also be created. The role of DRBD is to establish synchronization between two systems and allows the replication of data These are the steps we are going to follow to achieve synchronization:

  

### Step-1: Establishing connection
New hostname
`# hostnamectl set-hostname`  
 
To give permissions
`# su ` 
 
We have two  **CentOS 7**  machines,  `mr.server`  and  `mr.client`  . First we will resolve them for DNS in order to communicate with each other.
On both machines type: 
 `# vim /etc/hosts `

Now save the files and type below commands on each machine to check if they are connected
`# ping mr.server`  
  `# ping mr.client`


### Step-2: Flushing the IP-Tables
Now we have to flush ip tables on both machines by typing below command:
`# iptables -F`


To check output type the below command.
`# iptables -L`



### Step-3: Checking NTP service
Now we will check if NTP(Network time Protocol) is installed and working properly on both machines or not because it is very important that both servers shall be synchronized with each other.

To check if NTP service is installed or not
`# rpm -qa | grep ntp ` 
 
To install NTP if it is not installed
`# yum install -y ntp` 

We will enable our machine ip for NTP request by changing the entry in the configuration file.
 `# vim /etc/ntp.conf`
 
Then restart the service by typing below command.
`# systemctl restart ntpd`

Now, type this command to allow the NTP incoming requests.
`# firewall-cmd --permanent --add-service=ntp`
`# firewall-cmd --reload`

### STEP 4: MAKING LVM FOR DRBD

**4a) Creating partitions for LVM**  First we have to create LVM partitions on both machines.

To check details of disk we have to type command:
`# fdisk -l` 

The second step will be to create partitions in sdb so we will do that by typing below command.
`# fdisk /dev/sdb `

**4b) Creating physical volume** For creating physical volume type the following command.
`# pvcreate /dev/sdb1 `

**4c) Creating Volume Group** for creating volume group type the following command
`# vgcreate vgdrbd /dev/sbd1` 

**4d) Creating Logical Volume** Now for creating logical volume we have type following command.
`# lvcreate -n lvdrbd /dev/mapper/vgdrbd -L 4000M`

### Step 5: INCLUDING ENTRIES IN SYSCTL CONFIG FILE

We will add few entries in the sysctl config file of both the machines by typing below command.
`# vi /etc/sysctl.conf ` 
 
Now look at the output by this command:
`# sysctl -p` 

Using scp command for configuration
`# scp /etc/sysctl.conf mr.client:/etc/`



### Step 6: INSTALLING AND CONFIGURING DRBD 
To install DRBD
 `rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org`
    
` rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm`

Installing kmod in both nodes:
`# yum install -y kmod-drbd84 drbd84-utils` 

Configuring the drbd file on both nodes:
`# vim /etc/drbd.conf`  

Start DRBD by using the following command:
`# modprobe drbd ` 

Now echo modprobe drd in rc.local by the following command:
`# echo "modprobe drbd" >> /etc/rc.local` 

Initialize the DRBD resource by this command:
`# drbdadm create-md resource0`

Now, enable the resource:
`# drbdadm up resource0`

Check the status of the drbd:
`# drbdadm status resource0`

On your first machine, start the initial full synchronization:
`# drbdadm primary --force resource0`

Now, check the status:
`# drbdadm status resource0`

Now create the filesystem:
`# mkfs.ext4 /dev/drbd0`

Mount the DRBD device on any directory, we are using /opt:
`# mount /dev/drbd0 /opt/`

Create some files inside /opt directory by using command:
`#cd /opt`
`# touch file1 file2 file3`

Now, on the first machine, unmount the DRBD and make it secondary:

```
cd
umount /opt
drbdadm secondary resource0


```

On the second machine, make the second node primary with this command:
`# drbdadm primary resource0`

Now, mount the DRBD device on the /opt directory with the following command:
`# mount /dev/drbd0 /opt`

Type and run this command to print the list of files in the /opt directory:
`# ls /opt`

All the files stored in the DRBD device will be shown there. Here, the installation and configuration of DRBD is successful

### Step 7: INSTALLING AND CONFIGURING PACEMAKER/COROSYNC

Now, enable the High Availability Repo and install pacemaker/corosync:
`# yum install pacemaker corosync` 

If it is disabled so we will enable it by typing following command.

`# yum config-manager --set-enabled ha`

After that install the following package for HA cluster by typing following command.

`# yum install --assumeyes pacemaker corosync pcs`

   For cluster, we allow firewalls to enable traffic in cluster.

> Opened UDP port: 5404
>  Opened TCP port: 2224 

`# iptables -I -p tcp -m state --state NEW -m tcp --dport 2224 -j `

 To see  saved changes 
 `# service iptables save` 

  Starting pacemaker services
`# systemctl start pcsd`  

  For checking authenticating
`# pcs cluster auth mr.server mr.client`  

 Cluster setup, cluster name defined
 `# pcs cluster setup --name cluster mr.server mr.client`

Starting clusters
`# pcs cluster start --all` 

 Checking cluster status to check if devices online or not
 `# pcs status cluster` 
