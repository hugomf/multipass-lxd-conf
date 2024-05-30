## Multipass-LXC Host Network configuration


**Macvlan Network:** Allows containers to have their own MAC address, appearing as unique devices on the network.


* Create a multipass ubuntu instance and name it lxd-server

```shell
mp launch -n lxd-server -c 2 -m 4G -d 50G
```

Connect to lxd-server
```shell
mp shell lxd-server
```


* Initialize the configuration

```shell
sudo lxd init --auto --trust-password password --network-address '[::]'
```

* Load the macvlan kernel module
  
```shell
sudo modprobe macvlan
```



* create the macvlan network in lxd

```shell
# Verify the network configuration
ip addr  # check the interface id to change the parent flag in the next command

incus network create macvlan0 --type=macvlan parent=eth0
```

* In order to run amazonlinux instances we need cgroup v1, to do that we need to download the image and unset the requirements cgroup

```shell
lxc image copy images:amazonlinux local: --copy-aliases
lxc image unset-property amazonlinux requirements.cgroup
```


* Go back to your host machine
  
```shell
exit
```


* Add the remote server in the client
```shell
lxc remote add default $(mp info lxd-server | grep IPv4 | awk '{print $2}') --password password --accept-certificate
```

* Create an amazon instance
  
```shell
incus launch images:amazonlinux/2023/arm64 amazonlinux --network macvlan0
```


### Notes:

**Why macvlan?**

> **WiFi Limitations:** Bridging over **WiFi** is generally not recommended because many WiFi drivers do not support it. macvlan is a workaround that allows instances to communicate on the network as if they are separate physical devices.