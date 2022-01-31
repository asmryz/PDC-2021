
#About Project

An HPC cluster is a collection of many separate servers (computers), called nodes, which are connected via a fast interconnect. There may be different types of nodes for different types of tasks

#Steps

- Creation of VM in VMware Server
- Linux Installation on Nodes (ContOS)
- SSH equivalance esatblishment for user root
- SSH Hotkeys collection
- Installation iof PDSH
- Setup NFS 
- Creation of ordinary user and also setup SSH equivalance
- Installing of prerequisite packages
- Installination of MPI
- Compiling Linpack
- Becnhmarking




#Description

- Setup a Local machine on VM ware.- It should be a Cent OS Linux
- Size of Master Node should be 1024MB
- We will use Bridged Network



#Usage

###First check About this computer 

```bash
cat /etc/hostname
```

###Install emacs-nox

```bash
sudo yum install -y emacs-nox

sudo emacs /etc/hostname
```

###Open router from IP using browser

###Check MACADDRESS of PC by Command ifconfig

To install ssh on server computer

```bash
sudo dnf install openssh-server
```

###Check SSH is running

```bash
ps -A | grep sshd
```

###Run this command to check server is listining to right port

```bash
sudo ss -lnp | grep sshd
```

###Login to the machine that you are using

```bash
ssh -v localhost
```

###Copy id_rsa.pub file from this command we don't need to type password again

```bash
ssh -copy-id hostname@192.168.218.133
```

###It shows the lookup table of IP address and alias

```bash
cat /etc/hosts
```

###To login to there computer

```bash
ssh node1
```

###Install NFS on Master Node

```bash
sudo dnf install nfs-utils
```

###Install NFS on Client Node

```bash
sudo yum install nfs-utils
```

###In server computer

```bash
cd /
```

###Make a directory in root

```bash
sudo mkdir /nfs
```

###Open Editor

```bash
sudo vi /etc/exports
```

###Run this command

```bash
/nfs *(rw,sync)
```

###Restart Server

```bash
sudo systemctl restart --now nfs-server
```

###Create NFS Directory in Client

```bash
sudo mkdir /nfs
```

###Client Computer

```bash
cat /etc/fstab
```

###Edit fstab directory

```bash
sudo emacs /etc/fstab
```

###CLient Computer

```bash
Root:/nfs /nfs nfs
```

###Check Owner of Particular file System

```bash
ls -ld /nfs
```

###Change owner name

```bash
sudo chown anas /nfs
```

###Owner Changed

```bash
ls -ld /nfs
```
