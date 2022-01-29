<div id="top"></div>



<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
    <img src="images/logo.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">Best-README-Template</h3>

  <p align="center">
    An awesome README template to jumpstart your projects!
    <br />
    <a href="https://github.com/othneildrew/Best-README-Template"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/othneildrew/Best-README-Template">View Demo</a>
    ·
    <a href="https://github.com/othneildrew/Best-README-Template/issues">Report Bug</a>
    ·
    <a href="https://github.com/othneildrew/Best-README-Template/issues">Request Feature</a>
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
HA Cluster with DRBD, Pacemaker & Corosync

HA stands for High Availability. The purpose of HA Cluster is that it provides 24/7 availability of services that are running on our server and does not cause any downtime. This is done with the help of more than 1 server so that if one of the servers goes down then the services running on that server are automatically started on another server.

<p align="right">(<a href="#top">back to top</a>)</p>



### Built With

We build our HA clusters with the help of these below CRMs:

* https://clusterlabs.org/
* http://corosync.github.io/corosync/
* https://www.nginx.com/

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Getting Started

Pacemaker and Corosync will be used to configure the HA Cluster. Pacemaker is a resource management that has the ability to start and stop resources. Corosync is a messaging component that is in charge of communication between cluster nodes (servers).

We will utilise DRBD, which stands for Distributed Replicated Block Device, to maintain the filesystems synced across all nodes in the cluster.


### Prerequisites

Below are the steps one by one how we configured our cluster
* npm
  ```sh
  vi /etc/hosts
  ```

### Installation


Configuration

First, go to the hosts file and add the domain name against the IPs of the servers to locally resolve the IPs on all the nodes.

# vi /etc/hosts
192.168.146.128 vm1
192.168.146.130 vm2
Then, we need to flush all the chains, which will delete all the firewall rules previously defined.

# iptables -F
Now we will configure NTP (Network Time Protocol) so that every node in the cluster is configured with the same time.

On vm1, install chrony and configure it to act as NTP Server:

# dnf install chrony
# systemctl enable chronyd
# vi /etc/chrony.conf
allow 192.168.0.0/16
# systemctl restart chronyd
# firewall-cmd --permanent --add-service=ntp
# firewall-cmd --reload
On vm2, install chrony and configure it to act as NTP Client:

# dnf install chrony
# systemctl enable chronyd
# vi /etc/chrony.conf
Server 192.168.148.128
# systemctl restart chronyd
# firewall-cmd --permanent --add-service=ntp
# firewall-cmd --reload
Now, we will create LVM partition on both nodes:

# fdisk /dev/sdb
# pvcreate /dev/sdb1
# vgcreate vgdrbd /dev/sdb1
# lvcreate -L +4000M -n lvdrbd /dev/mapper/vgdrbd
Now, we will add some IPv4 settings on both nodes:

# vi /etc/sysctl.d/ipv4settings.conf
net.ipv4.conf.ens33.arp_ignore = 1

net.ipv4.conf.all.arp_announce = 2

net.ipv4.conf.ens33.arp_announce = 2
# sysctl -p
Then, we will configure DRBD on both nodes. First, install drbd from the following link:

# dnf install -y https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/d/drbd-9.17.0-1.el8.x86_64.rpm
Then, install drbd-utils from this link:

# dnf install -y https://download-ib01.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/d/drbd-utils-9.17.0-1.el8.x86_64.rpm
After that, install kmod-drbd:

# dnf install -y https://mirror.rackspace.com/elrepo/elrepo/el8/x86_64/RPMS/kmod-drbd90-9.1.5-1.el8_5.elrepo.x86_64.rpm
Now, we will add some configuration in the DRBD configuration file on both nodes:

# vi /etc/drbd.conf
resource r0 {
  on vm1 {
    device    /dev/drbd0;
    disk      /dev/vgdrbd/lvdrbd;
    address   192.168.103.128:7789;
    meta-disk internal;
  }
  on vm2 {
    device    /dev/drbd0;
    disk      /dev/vgdrbd/lvdrbd;
    address   192.168.103.130:7789;
    meta-disk internal;
  }
}
And add this configuration in the DRBD global configuration file on both nodes:

# vi /etc/drbd.d/global_common.conf
global {
   usage-count yes;
}
common {
 net {
   protocol C;
 }
}
Enable DRBD service port 7789 on the firewall on both nodes:

# firewall-cmd --add-port=7789/tcp --permanent
# firewall-cmd --reload
Now we initialize and enable the DRBD resource on both nodes:

# drbdadm create-md r0
# drbdadm up r0
# drbdadm status r0
Now we will configure primary DRBD server. This command should be run on any one node:

# drbdadm primary --force r0
This will start the synchronization. It will take some time.

After the synchronization is complete, we will create a filesystem on the DRBD device on primary node, create a directory on both nodes and mount the DRBD device to that directory on primary node:

# mkfs.ext3 /dev/drbd0
# mkdir /data/
# mount /dev/drbd0 /data/
Now we will move on to install and configure Pacemaker on both nodes:

# dnf config-manager --set-enabled HighAvailability
# dnf install pcs pacemaker fence-agents-all -y
Then, a group named hacluster will be created. We will configure password for this group on both nodes:

# passwd hacluster
After that, we configure the firewall:

# firewall-cmd --permanent --add-service=high-availability
# firewall-cmd --reload
Now, on any one of the cluster node, authenticate as the hacluster user with the following command:

# pcs host auth vm1 vm2
Now we will create a HA cluster with 2 nodes with the name my_cluster:

# pcs cluster setup my_cluster vm1 vm2
Now we will start the cluster:

# pcs cluster start --all
Now, check the status of corosync across cluster nodes:

# pcs status corosync
To check the rest of the stack:

# pcs status
Also enable corosync and pacemaker to automatically start on boot:

# systemctl enable corosync
# systemctl enable pacemaker
As we do not have any STONITH device to work with, we will disable it.

# pcs property set stonith-enabled=false
Next, we will create a heartbeat resource for DRBD Cluster File System:

# pcs resource create fs_drbd ocf:heartbeat:Filesystem device=/dev/drbd0 directory=/data fstype=ext3
# pcs resource status
In order to verify our DRBD configuration, we will halt the primary node in which the DRBD filesystem is mounted currently and check if the filesystem automatically starts on the other node or not:

# pcs node standby vm1
# pcs resource status
By checking the status, if the DRBD filesystem is automatically started on the other node then verification was successful.

Now we will configure Nginx web server in our cluster. First we will install nginx on both nodes:

# yum install epel-release -y
# yum install nginx -y
In order to modify the default page of nginx, we will do the following:

On vm1:

# echo '<h1>VM1</h1>' > /usr/share/nginx/html/index.html
On vm2:

# echo '<h1>VM2</h1>' > /usr/share/nginx/html/index.html
Create 2 new resources of floating IP address with name 'virtual_ip' and nginx web server with name 'webserver'. Floating IP is the IP address that will be moved automatically from one server to another server.

# pcs resource create virtual_ip ocf:heartbeat:IPaddr2 ip=192.168.146.131 cidr_netmask=32 op monitor interval=30s
# pcs resource create webserver ocf:heartbeat:nginx configfile=/etc/nginx/nginx.conf op monitor timeout="5s" interval="5s"
Now verify the status of the resources with the following command:

# pcs status resources
After that, we will add constraints and rules for the High Availability resources:

# pcs constraint colocation add webserver with virtual_ip INFINITY
This constraint specifies that the webserver and virtual_ip will run on the same node.

# pcs constraint order virtual_ip then webserver
This constraint specifies that first the virtual_ip will start then the webserver will start.

Restart the cluster.

# pcs cluster stop --all
# pcs cluster start --all
Add firewall rule for http.

# firewall-cmd --add-service=http --permanent
# firewall-cmd --reload
Now, access 192.168.103.131 in the web browser to see the default page being served from the primary node. If we halt the primary node and then access the IP, then the default page should be served from the other node. This indicated that our high availability nginx web server is working perfectly.
    ```
<p align="right">(<a href="#top">back to top</a>)</p>



<!-- USAGE EXAMPLES -->
## Usage
This project was used to access the website with virtual IP address.

<p align="right">(<a href="#top">back to top</a>)</p>



<!-- ROADMAP -->
## Roadmap

- [ ] Multi-language Support
    - [ ] English

<p align="right">(<a href="#top">back to top</a>)</p>


<!-- CONTACT -->
## Contact

```sh
Ali Bin Khalid 1812219 - cs1812219@szabist.pk
Zohaib M Ali 1812252 - cs1812252@szabist.pk
Rana Omer 1812244 - cs1812244@szabist.pk
Uneeb Ahmed 1812251 - cs1812251@szabist.pk
```
<p align="right">(<a href="#top">back to top</a>)</p>


