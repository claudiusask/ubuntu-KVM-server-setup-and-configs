# ubuntu-KVM-server-setup-and-configs
All the setup and configs for Ubuntu 20.04


Do the following in Ubuntu 20.04 with GUI or Headless. I am using headless.

``` sudo apt -y install bridge-utils cpu-checker libvirt-clients libvirt-daemon qemu qemu-kvm ```

Now go and check if the kvm is set with ```kvm-ok```

If headless use the cockpit. install cockpit with ```sudo apt install cockpit``` and then ```sudo systemctl start cockpit```

The cockpit uses port 9090 like this ```https://server-FQDN-OR-IP:9090``` might have to open port on ubuntu ```sudo ufw allow 9090/tcp```

Now use your systemAdmin to login in 

To view Virtual machines in cockpit webportal ```sudo apt install cockpit-machines -y```

Now when we installed the KVM, it came with one virbr0 which is NAT network which is ok for some tasks but if you need the bridge (meaning the vm will be on the same network as host machine) we have to create one bridge network.

Because ubuntu 20.04 uses netplan I went with the default netplan instaed of NetworkManager.

Go to /etc/netplan/00-installer-config.yaml (your might be different but it should be in the same location). 

Make a backup copy with adding .bak to the end of ```00-installer-config.yaml.bak```

The below network name ```eno1``` might be different in your system check with ```ip addr```

```	# This is the network config written by 'subiquity'
	network:
	  ethernets:
	    eno1:
	      dhcp4: no
	      dhcp6: no
	  version: 2
	
	  bridges:
	    br0:
	     interfaces: [eno1]
	     addresses: [10.0.0.10/24]
	     gateway4: 10.0.0.1
	     nameservers:
	       addresses:
	       - 10.0.0.1
	     parameters:
	       stp: false
	       forward-delay: 0
	     dhcp4: no
	     dhcp6: no
```

Now we need to add this bridge network into KVM environment.

Create a new file ```/etc/libvirt/qemu/networks/``` with the name bridge.xml and place the following in it:

```
<network>
  <name>br0</name>
  <forward mode="bridge"/>
  <bridge name="br0" />
</network>
```

After creating this file go to cli and run ```sudo virsh net-define bridge.xml```

```sudo virsh net-start br0```

```sudo virsh net-autostart br0```

Now run ```sudo virsh net-list --all``` to view the newly created bridge network ready to be served to VM's
