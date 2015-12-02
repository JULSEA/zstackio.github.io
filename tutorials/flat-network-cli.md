---
title: ZStack Tutorials
layout: tutorialDetailPage
sections:
  - id: overview 
    title: Overview
  - id: prerequisites 
    title: Prerequisites
  - id: login 
    title: LogIn
  - id: createZone 
    title: Create Zone
  - id: createCluster 
    title: Create Cluster
  - id: addHost 
    title: Add Host
  - id: addPrimaryStorage
    title: Add Primary Storage
  - id: addBackupStorage
    title: Add Backup Storage
  - id: addImage
    title: Add Image
  - id: createL2Network
    title: Create L2 Network
  - id: createL3Network
    title: Create L3 Network
  - id: createInstanceOffering
    title: Create Instance Offering
  - id: createVirtualRouterOffering
    title: Create Virtual Router Offering
  - id: createVM
    title: Create Virtual Machine
---

### Flat Network

<h4 id="overview">1. Overview</h4>
<img src="/images/flat_network.png">

Flat networks are very popular for small businesses and internal networks which cannot be accessed by traffics out of data center.
A flat network is very easy to deploy, usually has only one L2 broadcast domain, and machines on it can
access the internet through the core router of the data center. In this example, we assume you already have a subnet that has been routed to the internet;
we will create a few VMs and assign them IPs from that subnet.

<hr>

<h4 id="prerequisites">2. Prerequisites</h4>

We assume you have followed [Quick Installation Guide](../installation/index.html) to install ZStack on a single Linux machine, and
the ZStack management node is up and running. To use the command line tool, type below command in your shell terminal:

    #zstack-cli
    
<img class="img-responsive" src="/images/tutorials/t1/zstackCli.png">

<div class="bs-callout bs-callout-info">
  <h4>Connect to a remote management node</h4>
  By default, zstack-cli connects to the ZStack management node on the local machine. To connect
  to a remote node, using option '-H ZSTACK_NODE_HOST_IP'; for example: zstack-cli -H 192.168.0.224
</div>

To make things simple, we assume you have only one Linux machine with one network card that can access the internet; besides, there are
some other requirements:

+ At least 20G free disk that can be used as primary storage and backup storage
+ Several free IPs that can access the internet
+ NFS server is enabled on the machine (done in [Quick Installation Guide](../installation/index.html))
+ SSH credentials for user root (done in [Quick Installation Guide](../installation/index.html))

<div class="bs-callout bs-callout-info">
  <h4>Configure root user</h4>
  The KVM host will need root user credentials of SSH, to allow Ansible to install necessary packages and to give the KVM agent full control
  of the host. As this tutorial use a single machine for both ZStack management node and KVM host, you will need to configure credentials for
  the root user.
  
  <h5>CentOS:</h5>
  <pre><code>sudo su
passwd root</code></pre>

  <h5>Ubuntu:</h5>
  You need to also enable root user in SSHD configuration.
  <pre><code>1. sudo su
2. passwd root
3. edit /etc/ssh/sshd_config
4. comment out 'PermitRootLogin without-password'
5. add 'PermitRootLogin yes'
6. restart SSH: 'service ssh restart'</code></pre>
</div>

Based on those requirements, we assume below setup information:

+ ethernet device name: eth0
+ eth0 IP: 10.0.101.20
+ free IP range: 10.0.101.100 ~ 10.0.101.150 (these IPs can access the internet)
+ primary storage folder: 10.0.101.1:/home/nfs
+ backup storage folder: /home/sftpBackupStorage

<div class="bs-callout bs-callout-warning">
  <h4>Slow VM stopping due to lack of ACPID:</h4>
    Though we don't show the example of stopping VM, you may find stopping a VM takes more than 60s. That's 
    because the 15M ttylinux we use in the tutorial doesn't support ACPID that receives KVM's shutdown event, ZStack has to
    wait for 60 seconds timeout then destroy it. It's not a problem for regular Linux distributions which have ACPID installed.
</div>

<div class="bs-callout bs-callout-warning">
  <h4>Avoid DHCP conflict</h4>
  Please make sure you don't have a DHCP server in the network because ZStack will spawn its own DHCP server; if you have a
  DHCP server in the network and cannot remove it, please use an IP range that is unlikely used by your DHCP server, otherwise the VM 
  may not receive an IP from ZStack's DHCP server but from yours.
</div>

<hr>

<h4 id="login">3. LogIn</h4>

open zstack-cli and login with admin/password:

	>>> LogInByAccount accountName=admin password=password	

<img class="img-responsive" src="/images/tutorials/t1/cliLogin.png">

<hr>

<h4 id="createZone">4. Create Zone</h4>

create a zone with name 'ZONE1' and description 'zone 1':

	>>> CreateZone name=ZONE1 description='zone 1'

<img class="img-responsive" src="/images/tutorials/t1/cliCreateZone.png">

<div class="bs-callout bs-callout-info">
  <h4>Substitute your UUIDs for those in this tutorial</h4>
  Resources are all referred by UUIDs in CLI tutorials. The UUIDs are generated by ZStack when you create a resource, so they
  may vary from what you see in tutorials. UUIDs of resources will be printed out in a JSON object to the screen after resources
  are created; however, it's inconvenient to scroll up screen to find UUIDs of resources that are created very early. We add
  buttons <img src="/images/tutorials/find-uuid.png" style="border:none"> to sections, which will show you commands of retrieving UUIDs of resources,
  so please make sure you replace UUIDs in tutorials with yours.
</div>

<hr>

<h4 id="createCluster">5. Create Cluster</h4>

create a cluster with name 'CLUSTER1' and hypervisorType 'KVM' under zone 'ZONE1':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#5">Find UUID</button>

<div id="5" class="collapse">
<pre><code>QueryZone fields=uuid, name=ZONE1</code></pre>
</div>

	>>> CreateCluster name=CLUSTER1 hypervisorType=KVM zoneUuid=69b5be02a15742a08c1b7518e32f442a

<img class="img-responsive" src="/images/tutorials/t1/cliCreateCluster.png">

<hr>

<h4 id="addHost">6. Create Host</h4>

add KVM Host 'HOST1' under 'CLUSTER1' with correct host IP address and root username and password:

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#6">Find UUID</button>

<div id="6" class="collapse">
<pre><code>QueryCluster fields=uuid, name=CLUSTER1</code></pre>
</div>

	>>> AddKVMHost name=HOST1 managementIp=10.0.101.20 username=root password=password clusterUuid=2e88755b7dd0411f9dfc5362fc752b88

<img class="img-responsive" src="/images/tutorials/t1/cliCreateHost.png">

<div class="bs-callout bs-callout-warning">
  <h4>A little slow when first time adding a host</h4>
  It may take a few minutes to add a host because Ansible will install all dependent packages, for example, KVM, on the host.
</div>

<hr>

<h4 id="addPrimaryStorage">7. Add Primary Storage</h4>

add Primary Storage 'PRIMAYR-STORAGE1' with NFS URI '10.0.101.20:/usr/local/zstack/nfs_root' under zone 'ZONE1':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#7">Find UUID</button>

<div id="7" class="collapse">
<pre><code>QueryZone fields=uuid, name=ZONE1</code></pre>
</div>

	>>> AddNfsPrimaryStorage name=PRIMARY-STORAGE1 url=10.0.101.20:/usr/local/zstack/nfs_root zoneUuid=69b5be02a15742a08c1b7518e32f442a

<img class="img-responsive" src="/images/tutorials/t1/cliAddPrimaryStorage.png">

<div class="bs-callout bs-callout-info">
  <h4>Format of NFS URL</h4>
  The format of URL is exactly the same to the one used by Linux <i>mount</i> command.
</div>

<hr>

attach 'PRIMARY-STORAGE1' to 'CLUSTER1':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#7_1">Find UUID</button>

<div id="7_1" class="collapse">
<pre><code>QueryCluster fields=uuid, name=CLUSTER1</code></pre>
<pre><code>QueryPrimaryStorage fields=uuid, name=PRIMARY-STORAGE1</code></pre>
</div>

	>>> AttachPrimaryStorageToCluster primaryStorageUuid=35405cbbb25d497c94b8484e487f2496 clusterUuid=2e88755b7dd0411f9dfc5362fc752b88

<img class="img-responsive" src="/images/tutorials/t1/cliAttachPrimaryStorageToCluster.png">

<hr>

<h4 id="addBackupStorage">8. Add Backup Storage</h4>

add sftp Backup Storage 'BACKUP-STORAGE1' with its IP address('10.0.101.20'), root username('root'), password('password') and sftp folder path('/home/sftpBackupStorage'):

	>>> AddSftpBackupStorage name=BACKUP-STORAGE1 hostname=10.0.101.20 username=root password=password url=/home/sftpBackupStorage

<img class="img-responsive" src="/images/tutorials/t1/cliAddBackupStorage.png">

<hr>

attach new created Backup Storage('BACKUP-STORAGE1') to zone('ZONE1'):

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#8">Find UUID</button>

<div id="8" class="collapse">
<pre><code>QueryZone fields=uuid, name=ZONE1</code></pre>
<pre><code>QueryBackupStorage fields=uuid, name=BACKUP-STORAGE1</code></pre>
</div>

	>>> AttachBackupStorageToZone backupStorageUuid=e5dfe0824d8a4503bbc1b6b51782b5a3 zoneUuid=69b5be02a15742a08c1b7518e32f442a

<img class="img-responsive" src="/images/tutorials/t1/cliAttachBackupStorageToZone.png">

<hr>

<h4 id="addImage">9. Add Image</h4>

add Image('ttylinux') with format 'qcow2', 'RootVolumeTemplate' type, 'Linux' platform and image URL('http://download.zstack.org/templates/ttylinux.qcow2') to backup storage ('BACKUP-STORAGE1'):

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#9_1">Find UUID</button>

<div id="9_1" class="collapse">
<pre><code>QueryBackupStorage fields=uuid, name=BACKUP-STORAGE1</code></pre>
</div>

<div class="bs-callout bs-callout-success">
  <h4>Fast link for users of Mainland China</h4>
  由于国内访问我们位于美国的服务器速度较慢，国内用户请使用以下链接：
  
  <pre><code>http://7xi3lj.com1.z0.glb.clouddn.com/templates/ttylinux.qcow2</code></pre>
</div>

	>>> AddImage name=ttylinux format=qcow2 mediaType=RootVolumeTemplate platform=Linux url=http://download.zstack.org/templates/ttylinux.qcow2 backupStorageUuids=e5dfe0824d8a4503bbc1b6b51782b5a3

<img class="img-responsive" src="/images/tutorials/t1/cliAddImage.png">

this image will be used as user VM image.

<hr>

add another Image('VIRTUAL-ROUTER') with format 'qcow2', 'RootVolumeTemplate' type, 'Linux' platform and image URL({{site.vr_en}}) to backup storage ('BACKUP-STORAGE1'):

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#9_2">Find UUID</button>

<div id="9_2" class="collapse">
<pre><code>QueryBackupStorage fields=uuid, name=BACKUP-STORAGE1</code></pre>
</div>

<div class="bs-callout bs-callout-success">
  <h4>Fast link for users of Mainland China</h4>
  由于国内访问我们位于美国的服务器速度较慢，国内用户请使用以下链接：
  
  <pre><code>{{site.vr_ch}}</code></pre>
</div>

	>>> AddImage name=VIRTUAL-ROUTER format=qcow2 mediaType=RootVolumeTemplate platform=Linux url={{site.vr_en}} backupStorageUuids=e5dfe0824d8a4503bbc1b6b51782b5a3


<img class="img-responsive" src="/images/tutorials/t1/cliAddVRImage.png">

this image will be used as Virtual Router VM image.

<div class="bs-callout bs-callout-info">
  <h4>Cache images in your local HTTP server</h4>
  The virtual router image is about 432M that takes a little of time to download. We suggest you use a local HTTP server
  to store it and images created by yourself.
</div>

<hr>

<h4 id="createL2Network">10. Create L2 Network</h4>

create No Vlan L2 Network 'FLAT-L2' with physical interface as 'eth0' under 'ZONE1':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#10_1">Find UUID</button>

<div id="10_1" class="collapse">
<pre><code>QueryZone fields=uuid, name=ZONE1</code></pre>
</div>

	>>> CreateL2NoVlanNetwork name=FLAT-L2 physicalInterface=eth0 zoneUuid=69b5be02a15742a08c1b7518e32f442a

<img class="img-responsive" src="/images/tutorials/t2/cliCreateL2NoVlan.png">

<hr>

attach 'FLAT-L2' to 'CLUSTER1':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#10_2">Find UUID</button>

<div id="10_2" class="collapse">
<pre><code>QueryCluster fields=uuid, name=CLUSTER1</code></pre>
<pre><code>QueryL2Network fields=uuid, name=FLAT-L2</code></pre>
</div>

	>>> AttachL2NetworkToCluster l2NetworkUuid=7a7d82f840da4fe6855546befd666b99 clusterUuid=2e88755b7dd0411f9dfc5362fc752b88

<img class="img-responsive" src="/images/tutorials/t2/cliAttachL2NoVlanToCluster.png">

<h4 id="createL3Network">11. Create L3 Network</h4>

create 'FLAT-L3' L3 network on FLAT-L2 L2 network:

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#11_1">Find UUID</button>

<div id="11_1" class="collapse">
<pre><code>QueryL2Network fields=uuid, name=FLAT-L2</code></pre>
</div>

	>>> CreateL3Network name=FLAT-L3 l2NetworkUuid=7a7d82f840da4fe6855546befd666b99 dnsDomain=tutorials.zstack.org

<img src="/images/tutorials/t2/cliCreateL3Network.png">

<hr>

create IP Range for 'FLAT-L3':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#11_2">Find UUID</button>

<div id="11_2" class="collapse">
<pre><code>QueryL3Network fields=uuid, name=FLAT-L3</code></pre>
</div>

	>>> AddIpRange name=FLAT-IP-RANGE l3NetworkUuid=8677b5e8d66c477a9b34fbf5bbef84d5 startIp=10.0.101.100 endIp=10.0.101.150 netmask=255.255.255.0 gateway=10.0.101.1

<img class="img-responsive" src="/images/tutorials/t2/cliAddIpRange.png">

<hr>

add DNS for 'FLAT-L3':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#11_3">Find UUID</button>

<div id="11_3" class="collapse">
<pre><code>QueryL3Network fields=uuid, name=FLAT-L3</code></pre>
</div>

	>>> AddDnsToL3Network l3NetworkUuid=8677b5e8d66c477a9b34fbf5bbef84d5 dns=8.8.8.8

<img class="img-responsive" src="/images/tutorials/t2/cliAddDns.png">

<hr>

we need to get UUIDs of available network service providers, before attaching virtual router services to L3 network:

	>>> QueryNetworkServiceProvider

<img class="img-responsive" src="/images/tutorials/t1/cliQueryNetworkServiceProvider.png">

there are 2 available network service providers. In this tutorial, we just need the 'Virtual Router', which could provide 'DHCP', 'SNAT', 'DNS', 'PortForwarding' and 'Eip'.

<hr>

attach VirtualRouter services 'DHCP' and 'DNS' to 'FLAT-L3':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#11_6">Find UUID</button>

<div id="11_6" class="collapse">
<pre><code>QueryL3Network fields=uuid, name=FLAT-L3</code></pre>
<pre><code>QueryNetworkServiceProvider fields=uuid, name=VirtualRouter</code></pre>
</div>

	>>> AttachNetworkServiceToL3Network networkServices="{'96c5fbe222ad4b6586d35086b67ec07a':['DHCP','DNS']}" l3NetworkUuid=8677b5e8d66c477a9b34fbf5bbef84d5

<img class="img-responsive" src="/images/tutorials/t2/cliAttachNetworkServiceToL3.png">

<div class="bs-callout bs-callout-info">
  <h4>Structure of parameter networkServices</h4>
  It's a JSON object of map that key is UUID of network service provider and value is a list of network service types.
</div>

<hr>

<h4 id="createInstanceOffering">12. Create Instance Offering</h4>

create a guest VM instance offering 'small-instance' with 1 512Mhz CPU and 128MB memory:

	>>> CreateInstanceOffering name=small-instance cpuNum=1 cpuSpeed=512 memorySize=134217728

<img class="img-responsive" src="/images/tutorials/t1/cliCreateInstanceOffering.png">

<hr>

<h4 id="createVirtualRouterOffering">13. Create Virtual Router Offering</h4>

create a Virtual Router VM instance offering 'VR-OFFERING' with 1 512Mhz CPU, 512MB memory, management L3 network 'FLAT-L3', public L3 network 'FLAT-L3', image 'VIRTUAL-ROUTER' and isDefault 'True':

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#13">Find UUID</button>

<div id="13" class="collapse">
<pre><code>QueryImage fields=uuid, name=VIRTUAL-ROUTER</code></pre>
<pre><code>QueryL3Network fields=uuid,name, name=FLAT-L3</code></pre>
<pre><code>QueryZone fields=uuid, name=ZONE1</code></pre>
</div>

	>>> CreateVirtualRouterOffering name=VR-OFFERING cpuNum=1 cpuSpeed=512 memorySize=536870912 imageUuid=854801a869e149b092281e0ef65585f9 managementNetworkUuid=8677b5e8d66c477a9b34fbf5bbef84d5 publicNetworkUuid=8677b5e8d66c477a9b34fbf5bbef84d5 isDefault=True zoneUuid=69b5be02a15742a08c1b7518e32f442a

<img class="img-responsive" src="/images/tutorials/t2/cliCreateVirtualRouterOffering.png">

<hr>

<h4 id="createVM">14. Create Virtual Machine</h4>

create a new guest VM instance with instance offering 'small-instance', image 'ttylinux', L3 network 'FLAT-L3', name 'VM1' and hostname 'vm1'

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#14">Find UUID</button>

<div id="14" class="collapse">
<pre><code>QueryInstanceOffering fields=uuid, name=small-instance</code></pre>
<pre><code>QueryImage fields=uuid, name=ttylinux</code></pre>
<pre><code>QueryL3Network fields=uuid, name=FLAT-L3</code></pre>
</div>

	>>> CreateVmInstance name=VM1 instanceOfferingUuid=328d52eae4ff4ba0a685101c3116020a imageUuid=62cf76d08c944288a92de98af1405289 l3NetworkUuids=8677b5e8d66c477a9b34fbf5bbef84d5 systemTags=hostname::vm1

<img class="img-responsive" src="/images/tutorials/t2/cliVmCreation.png">

<div class="bs-callout bs-callout-warning">
  <h4>The first user VM takes more time to create</h4>
  For the first user VM, ZStack needs to download the image from the backup storage to the primary storage and create a virtual router VM on
  the private L3 network, so it takes about 1 ~ 2 minutes to finish.
</div>

from VM creation result, you can get the new created VM IP address is 10.0.101.124

<hr>

once the VM is created successfully, you can ssh to Login onto it through any machine that can reach subnet 10.0.101.0/24 and use 'ifconfig' to show the IP address.

<button type="button" class="btn btn-primary" data-toggle="collapse" data-target="#14_1">Find IP</button>

<div id="14_1" class="collapse">
<pre><code>QueryVmNic fields=ip vmInstance.name=VM1</code></pre>
</div>

	#ssh root@10.0.101.124

<img src="/images/tutorials/t2/cliSshVm1.png">

<hr>

you can also reach internet in the VM:

	# ssh root@10.0.101.124 "/sbin/ifconfig;ping -c3 www.alibaba.com"

<img src="/images/tutorials/t2/cliSshVm2.png">

### Summary

In this tutorial, we showed you how to create a flat network in ZStack. For more details about ZStack's L3 network, visit
[L3 Network in user manual](http://zstackdoc.readthedocs.org/en/latest/userManual/l3Network.html).
