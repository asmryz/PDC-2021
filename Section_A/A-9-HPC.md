Creation of VM in VMware Server

Linux Installation on Nodes (ContOS)

SSH equivalance esatblishment for user root

SSH Hotkeys collection

Installation iof PDSH

Setup NFS

Creation of ordinary user and also setup SSH equivalance

Installing of prerequisite packages

Installination of MPI

Compiling Linpack

Becnhmarking





Setup a Local machine on VM ware.- It should be a Cent OS Linux

Size of Master Node should be 1024MB

We will use Bridged Network



etc host file configuration

masternode 10.0.0.20

node1 10.0.0.11

node2 10.0.0.12

copy host files

scp /etc/hosts node1: /etc/

scp /etc/hosts node2: /etc/

generating key

ssh-keygen -t dsa

ssh-keygen -t rsa

cd .ssh/

ls -l

copy files to all nodes

scp -r .ssh node1: /root/

scp -r .ssh node2	: /root/









First check About this computer

cat /etc/hostname in terminal

sudo yum install -y emacs-nox

sudo emacs /etc/hostname

open router from IP using browser

check MACADDRESS of PC by Command ifconfig

run this command sudo dnf install openssh-server

ps -A | grep sshd (run this command to check SSH in running)

sudo ss -lnp | grep sshd (run this command to check server is listining to right port)

ssh -v localhost (to login to the machine that you are using)

ssh -copy-id hostname@IP Address of computer you want to login (by this command id\_rsa.pub file will be copied to guest and we donst want to again type password to login to guest from host)

cat /etc/hosts (it shows the lookup table of IP address and alias)

ssh (guest name) to login to there pc

sudo dnf install nfs-utils (install this on server machine)

sudo yum install nfs-utils (install this on client machine)

cd / (in server computer)

sudo mkdir /nfs (this will make a directory in root named nfs)

sudo vi /etc/exports (to open editor)

/nfs \*(rw,sync) (type this command)

sudo systemctl restart --now nfs-server (to restart the server)

sudo mkdir /nfs (create nfs directory in client)

cat /etc/fstab (on client computer)

sudo emacs /etc/fstab (to edit fstab directory)

master\_node\_name:/nfs /nfs nfs (on client computer)

ls -ld /nfs (to check the owner of perticular file system)

sudo chown anas /nfs (change owner of nfs)

ls -ld /nfs (owner has been change to anas)

