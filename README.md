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
```
## create flavor, boot instance
```
openstack flavor create --ram 1024 --disk 5 --vcpus 1 comet_flavor
openstack image create --disk-format qcow2 --file xenial-server-cloudimg-amd64-disk1.img --public comet_image
nova boot --image comet_image --flavor comet_flavor --key-name comet_keypair --security-group comet_secgroup --nic net-name=comet_inet vm1
openstack floating ip create public
openstack server add floating ip vm1 172.24.4.8
openstack router add subnet comet_router comet_isubnet
cd .ssh/
chmod 600 comet_identity
cd ..
ip netns list
sudo ip netns exec qrouter-bfbb29f0-4184-4072-975a-dfec366ded20 ssh -i .ssh/comet_identity2 ubuntu@192.168.20.6
```
```
openstack network create comet_enet
openstack subnet create --network comet_enet --subnet-range 192.168.248.0/24 --dns-nameserver 8.8.4.4 --gateway 192.168.248.1 comet_esubnet 
openstack router set comet_router --external-gateway comet_enet
openstack floating ip create comet_enet
nova boot --image comet_image --flavor comet_flavor --key-name comet_keypair --security-group comet_secgroup --nic net-name=comet_inet vm2
```
