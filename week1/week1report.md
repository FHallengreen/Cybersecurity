# WEEK 1: Understanding the network + Attacker machine - Red

<a id="part1-step2"></a>
### PART 1 - Install "red" machine

1. Find the desired Vagrant VM Box to run. I went with [debian/buster64](https://app.vagrantup.com/debian/boxes/buster64). In order to setup the vagrant file, do `vagrant box add debian/buster64` (which only works with Vagrant installed), then select provider (in our case 2 for Virtualbox) and then `vagrant init debian/buster64` to create a vagrant file using the buster64 box.


2. Then edit the vagrantfile to and add the neccesary configuration. I added the vmbox debian/buster64 and named the box "red".
Additionally the network must be configured according to the assignment, which mentions that the machine should be part of the yellow network. Therefore add a hostonly adapter and assign it to an IP-address in the range (192.168.100.**0-254**). I decided to assign it to .10.
When added, the vagrantfile should look as follows (when removing all the comments):
```
Vagrant.configure("2") do |config|
config.vm.box = "debian/buster64"
config.vm.define "red"
config.vm.network "private_network", type: "hostonly", ip: "192.168.100.10", name: "VirtualBox Host-Only Ethernet Adapter #2"
end
```

3. Type `vagrant up` to start up the VM.


4. Connect to the VM using `ssh vagrant red`. To check that the VM is connected to the internet i did `ping google.com`:

```
vagrant@buster:~$ ping google.com
PING google.com (142.251.36.14) 56(84) bytes of data.
64 bytes from ams15s44-in-f14.1e100.net (142.251.36.14): icmp_seq=1 ttl=56 time=33.5 ms
64 bytes from ams15s44-in-f14.1e100.net (142.251.36.14): icmp_seq=2 ttl=56 time=31.0 ms
64 bytes from ams15s44-in-f14.1e100.net (142.251.36.14): icmp_seq=3 ttl=56 time=28.4 ms
^C
--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 9ms
rtt min/avg/max/mdev = 28.360/30.964/33.534/2.112 ms
```

<a id="part2"></a>
## PART 2 - Network Analysis and Vulnerability Assessment

### 1. What did you have to configure on your red machine to have internat and to properly ping the web machine?

To ensure that the "red" machine had internet access and could properly ping the web, I configured its network settings in the Vagrantfile during setup ([See part 1, Step 2](#part1-step2)) and then did `ping google.com` when connected to the VM.

### 2. - What is the default gateway of each machine?

To find the default gateway i used the `route -n` and `ip route show` commands, which provides information about the route to access the internet. This is useful to understand the network topology.

<mark> **Important**</mark> -
As the isprouter doesn't have a default route to companyrouter, it needs to be configured first using: 
```
sudo ip route add 172.30.0.0/16 via 192.168.100.253 dev eth1
```

The default gateway for "red" machine is configured via NAT and shows as 10.0.2.2. Below is a table showing all gateways for the VM's.

|VM Machine| gateway|
| :--- | ---:|
|red| 10.0.2.2|
|isprouter| 10.0.2.2|
|companyrouter | 192.168.100.254|
|dc| 172.30.255.254|
|database| 172.30.255.254|
|win10| 172.30.255.254|
|web| 172.30.255.254|



### 3. - What is the DNS server of each machine?

For the "red" attacker machine, the DNS server is set to 10.0.2.3. I found this information by running the `cat /etc/resolv.conf` command. For Windows `ipconfig /all`

|VM Machine| DNS nameserver|
| :--- | ---:|
|red| 10.0.2.3|
|isprouter| 10.0.2.3|
|companyrouter| 10.0.2.3|
|dc| 127.0.0.1|
|database| 172.30.0.4|
|win10| 172.30.0.4|
|web| 172.30.0.4|


This nameserver is responsible for resolving domain names into IP addresses and is vital for the machine's ability to access resources on the internet.

### 4. - Which machines have a static IP and which use DHCP?

All machines are using a static IP except win10 which is using DHCP for a dynamic IP assigment.

### 5. - What routes should be configured and where, how do you make it persistent?

As mentioned in [part 2 (important)](#part2), the route is not set up by default and needs to be configured on each start.
Therefore adding a script on start-up is ideal to automate this process.
1. To install nano on Alpine Linux 3.0 (which isprouter is run on) use `sudo apk add nano` .
2. Then create a new script in the `/etc/local.d` folder by typing `sudo nano /etc/local.d/set-route.start`. Then add `ip route add 172.30.0.0/16 via 192.168.100.253 dev eth1` to the file, save it by using ctrl + x and pressing save.
3. In order for the script to be executeable type `sudo chmod +x /etc/local.d/set-route.start`  and afterwards enter `sudo rc-update add local default` to make the script run on boot.
4. Restart the VM using `sudo reboot` and once restarted, validate the route is added to the boot process by pinging companyrouter `ping 172.30.255.254`.

### 6. - Which users exist on which machines?

### 7. - What is the purpose (which processes or packages for example are essential) of each machine?

### 8. - Investigate whether the DNS server of the company network is vulnerable to a DNS Zone Transfer "attack" as discussed above...

