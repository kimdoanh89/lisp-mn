# lisp-mn
## create project, user, role
```
openstack project create --domain default --description 'COMET project' comet
openstack user create --password admin comet_admin
openstack role add --project comet --user comet_admin admin
```
## create internal network
```
openstack network create comet_inet
openstack subnet create --network comet_inet --subnet-range 192.168.20.0/24 comet_isubnet
```
## create router
```
openstack router create comet_router
openstack router set comet_router --external-gateway public
```
## create floating ip, keypair, security group
```
openstack floating ip create public
openstack keypair create comet_keypair
cd .ssh/
nano comet_identity //paste keypair that copied from previous step
cd ..
```
## create security group
```
openstack security group create comet_secgroup
```
add security rule to the security group
```
openstack security group rule create --remote-ip 0.0.0.0/0 --protocol tcp --dst-port 22 --ingress comet_secgroup
openstack security group rule create --remote-ip 0.0.0.0/0 --protocol icmp --ingress comet_secgroup
openstack security group rule create --remote-ip 0.0.0.0/0 --protocol icmp --egress comet_secgroup
```
## create flavor, boot instance
```
openstack flavor create --ram 1024 --disk 5 --vcpus 1 comet_flavor
openstack image create --disk-format qcow2 --file xenial-server-cloudimg-amd64-disk1.img --public comet_image
nova boot --image comet_image --flavor comet_flavor --key-name comet_keypair --security-group comet_secgroup --nic net-name=comet_inet vm1
```
## create and assign floating ip
```
openstack router add subnet comet_router comet_isubnet
openstack floating ip create public
openstack server add floating ip vm1 172.24.4.8
cd .ssh/
chmod 600 comet_identity
cd ..
```
## access the instance with ssh
```
ip netns list
sudo ip netns exec qrouter-bfbb29f0-4184-4072-975a-dfec366ded20 ssh -i .ssh/comet_identity2 ubuntu@192.168.20.6
```
```
openstack network create comet_enet
openstack subnet create --network comet_enet --subnet-range 192.168.248.0/24 --dns-nameserver 8.8.8.8 --gateway 192.168.248.2 comet_esubnet 
openstack router create router2
openstack router set router2 --external-gateway comet_enet
```
## create second internal network, vm2
```
openstack network create comet_inet2
openstack subnet create --network comet_inet2 --subnet-range 192.168.30.0/24 comet_isubnet2
openstack router add subnet router2 comet_isubnet2
nova boot --image comet_image --flavor comet_flavor --key-name comet_keypair --security-group comet_secgroup --nic net-name=comet_inet2 vm2
openstack floating ip create comet_enet
openstack server add floating ip vm2 192.168.248.5
sudo ip netns exec qrouter-17efa51e-42f3-4a8c-b38e-4ff264145a52 ssh -i .ssh/comet_identity2 ubuntu@192.168.30.9
```

## create third internal network, vm3
```
openstack network create comet_inet3
openstack subnet create --network comet_inet3 --subnet-range 192.168.40.0/24 comet_isubnet3
openstack router add subnet router2 comet_isubnet3
nova boot --image comet_image --flavor comet_flavor --key-name comet_keypair --security-group comet_secgroup --nic net-name=comet_inet3 vm3
openstack floating ip create comet_enet
openstack server add floating ip vm3 192.168.248.6
sudo ip netns exec qrouter-17efa51e-42f3-4a8c-b38e-4ff264145a52 ssh -i .ssh/comet_identity2 ubuntu@192.168.40.10
```
### create provider network
```
openstack network create  --share --external --provider-physical-network provider --provider-network-type flat provider
```

### adding dns 
The file /etc/resolv.conf is automatically overwritten. To prevent this, modify the base and update with sudo resolvconf -u
```
sudo nano /etc/resolvconf/resolv.conf.d/base
sudo resolvconf -u
sudo nano /etc/resolv.conf
```
the file as follows:
```
nameserver 143.53.133.130 143.53.133.132 8.8.4.4
search localdomain
```
Access the vm1
```
sudo ip netns exec qrouter-6c27dec6-485f-4f02-83d7-f237848f04e9 ssh -i .ssh/comet_identity3 ubuntu@10.0.0.6
```
Fix unable to resolve host. Two things to check (assuming your machine is called vm1, you can change this as appropriate):
- That the /etc/hostname file contains just the name of the machine.
- That /etc/hosts has an entry for localhost. It should have something like:
```
127.0.0.1 localhost
127.0.1.1 vm1
```

### Prepare the vm1 with oor, odl ...
Git clone oor and install Software Prerequisites
```
git clone https://github.com/OpenOverlayRouter/oor
sudo apt-get install build-essential git-core libconfuse-dev gengetopt libcap2-bin libzmq3-dev libxml2-dev
```
Install oor
```
cd oor
sudo make install
```

You can instruct your system to auto-configure these values during system
boot-up if you add the following lines to `/etc/sysctl.conf`. Remember to 
reboot your system after adding these lines.

    net.ipv4.conf.default.rp_filter=0
    net.ipv4.conf.all.rp_filter=0
    net.ipv4.ip_forward=1
    net.ipv6.conf.all.forwarding=1

Download and install ODL

    wget https://nexus.opendaylight.org/content/repositories/public/org/opendaylight/integration/karaf/0.8.2/karaf-0.8.2.zip
    sudo apt-get install unzip
    unzip karaf-0.8.2.zip
ODL needs JAVA to run. Download openjdk-8-jdk and set up the JAVA_HOME in `/etc/environment`

    sudo apt-get purge openjdk-\*
    sudo add-apt-repository ppa:openjdk-r/ppa  
    sudo apt-get update   
    sudo apt-get install openjdk-8-jdk
    
Add JAVA_HOME in `sudo gedit /etc/environment`    
        
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
Source `/etc/environment`

    source /etc/environment
    
### Prepare and create more instances
Acesss instance vm1
```
sudo ip netns exec qrouter-b571e023-f79d-44ad-906c-44a4597bf28d ssh -i .ssh/comet_identity ubuntu@10.0.0.5
```
Create `vm1_snapshot` image using Horizon
Create and launch vm2 for ODL
```
openstack network create private2
openstack subnet create --network private2 --subnet-range 11.0.0.0/24 private2_sub
openstack router add subnet router1 private2_sub
nova boot --image vm1_snapshot --flavor ds2G --key-name comet_keypair --security-group comet_secgroup --nic net-name=private2 vm2
openstack floating ip create public
openstack server add floating ip vm2 172.24.4.7
sudo ip netns exec qrouter-b571e023-f79d-44ad-906c-44a4597bf28d ssh -i .ssh/comet_identity ubuntu@11.0.0.11
```
Create and launch vm3 for server
```
openstack network create private3
openstack subnet create --network private3 --subnet-range 12.0.0.0/24 private3_sub
openstack router add subnet router1 private3_sub
nova boot --image vm1_snapshot --flavor ds1G --key-name comet_keypair --security-group comet_secgroup --nic net-name=private3 vm3
openstack floating ip create public
openstack server add floating ip vm3 172.24.4.21
sudo ip netns exec qrouter-b571e023-f79d-44ad-906c-44a4597bf28d ssh -i .ssh/comet_identity ubuntu@12.0.0.7
```
