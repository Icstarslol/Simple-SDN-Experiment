# Simple-SDN-Experiment
Project to create a SDN network and present functionalities of OpenFlow. Listed below are the steps taken to create the network in GNS3 and test the capabilities of the OpenFlow protocol so that the project is reproducable.

## Network Topology
<img width="759" alt="image" src="https://github.com/user-attachments/assets/013ab14e-97d3-4019-a894-e94127369b0d">
<br/><br/>

This network topology is centered around a Software-Defined Networking architecture, with the OpenDaylight controller (ODL-1) running the OpenFlow Manager application acting as the heart of the network. The ODL controller, with an IP address of 192.168.75.129, is connected to a central switch, enabling centralized control and management of the entire network. Switch1 serves as a hub, interconnecting various components, including the two Open vSwitch  instances (OVS-2, OVS-3) which act as software-defined bridges to extend the SDN's reach. These switches are connected to routers R1 and R2, which form the gateways for two distinct subnets, 10.1.1.1/24 and 10.1.1.2/24, respectively. R1 connects to Ubuntu-1 (192.168.10.100), while R2 connects to Ubuntu-2 (192.168.20.100). The topology also incorporates a NAT device, which plays the role of a DHCP server for the SDN devices. This setup aims to highlight SDN's pivotal role in simplifying network management by dynamically controlling traffic flow from the centralized SDN controller.

## Setting Up the Experiement Enviroment
In order to recreate the experiemnt enviroment, we need a two key software
* GNS3 with GNS VM
* Virtualisation Software (VMware Workstation / VirtualBox)

GNS3 provides a robust platform for designing, simulating, and testing complex network topologies. While virtualization software is required to host the GNS3 VM and the SDN Controller used in the experiment.

## Configuring the SDN Controller
The SDN Controller used in this experiment is an Ubuntu 20.04.6 LTS VM hosted in Virtual Box. We firstly install a few required packages
```bash
sudo apt update
sudo apt install -y software-properties-common git nodejs
```
Then we download and install the Java8 JDK
```bash
# Download the binary from https://www.oracle.com/sg/java/technologies/javase/javase8-archive-downloads.html
sudo tar -zxvf jdk-8u202-linux-x64.tar.gz -C /opt
nano ~./bashrc
export JAVA_HOME=/opt/jdk1.8.0_202
export PATH=$JAVA_HOME/bin:$PATH
source ~/.bashrc

# Alternatively using OpenJDK
sudo apt install openjdk-8-jre
nano ~./bashrc
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
source ~/.bashrc

# Verify that java is successfully installed
echo $JAVA_HOME
java -version
```
Next we get OpenFlow Manager to control the flows in the OVS switch tables
```bash
git clone https://github.com/CiscoDevNet/OpenDaylight-Openflow-App.git
cd OpenDayLight-OpenFlow-App

# Replace the base url line with the IP of the SDN Controller e.g. baseURL: "http://192.168.75.129:",
nano ./ofm/src/common/config/env.module.js

npm install -g grunt-cli
grunt

# We can access the application at http://<SDN Controller IP>:9000
```
Finally we install and configure OpenDaylight
```bash
wget https://nexus.opendaylight.org/content/groups/public/org/opendaylight/integration/distribution-karaf/0.3.0-Lithium/distribution-karaf-0.3.0-Lithium.tar.gz
tar zxvf distribution-karaf-0.3.0-Lithium.tar.gz
cd distribution-karaf-0.3.0-Lithium
./bin/karaf
feature:install odl-restconf-all odl-openflowplugin-all odl-l2switch-all odl-mdsal-all odl-yangtools-common
```
To integrate the SDN Controller VM into GNS3

*Edit -> Preferences -> VirtualBox -> VirtualBox VMs -> New*

Select the SDN Controller and import it

We then edit the network settings to connect the VM to GNS3

*Edit -> Preferences -> VirtualBox -> VirtualBox VMs -> Edit*

<img width="575" alt="Screenshot 2024-11-24 015109" src="https://github.com/user-attachments/assets/c89c336c-8fa9-4777-9be5-1d264051c76d">

## Configuring the Open vSwitch
For this experiement we will be using the Open vSwitch Management Appliance provided by GNS3

<img width="853" alt="image" src="https://github.com/user-attachments/assets/3b7a0b19-bd15-4d98-a353-70a0d65ce478">
<br/><br/>

Before turning on the switches. Enable the management interface (eth0) to use DHCP by right clicking the devices and selecting `Edit config`. The NAT cloud should assign an IP Address to the interface on the same subnet as the SDN Controller.
<br/><br/>

<img width="767" alt="Screenshot 2024-11-23 181651" src="https://github.com/user-attachments/assets/5b901db1-a4e0-4ce7-9fff-c7062a941d79">
<br/><br/>

After saving the initial configuration, we run the switches and configure them using the following commands
<br/><br/>

```
# Enable Spanning Tree Protocol to prevent loops
ovs-vsctl set Bridge br0 stp_enable=true

# Set the OpenFlow version to 1.3 [IMPORTANT otherwise ODL will not recognise the device]
ovs-vsctl set bridge br0 protocols=OpenFlow13

# View switch information e.g. dpid, no. of tables supported, no. of buffers, switch ports
ovs-ofctl show br0

# Set the SDN controller the switch will comunicate with (either ports 6633 or 6653 can be used)
ovs-vsctl set-controller br0 tcp:<controller_ip>:6633
ovs-vsctl show
```

## Configuring the Routers and Hosts
The routers are configured with the commands listed below to enable routing throughout the network

For R1:
```
# Configure interfaces
conf t
int f0/0
no shut
ip address 10.1.1.1 255.255.255.0
exit
int f0/1
no shut
ip address 192.168.10.1 255.255.255.0
exit

# Configure routing protocol
router rip
version 2
network 10.1.1.0
network 192.168.10.0
do sh ip route
exit
```
For R2:
```
# Configure interfaces
conf t
int f0/0
no shut
ip address 10.1.1.2 255.255.255.0
exit
int f0/1
no shut
ip address 192.168.20.1 255.255.255.0
exit

# Configure routing protocol
router rip
version 2
network 10.1.2.0
network 192.168.20.0
do sh ip route
exit
```
The two host machines are configured with static IP addresses and gateways pointing to their respective routers which can be done by right clicking the devices and selecting `Edit config`

<img width="429" alt="image" src="https://github.com/user-attachments/assets/1a2b1ca8-5f07-4e31-a28c-c45a0fd1efb6">

## Conducting the Experiemnt
Once the network is setup it is worth generating some network traffic to refresh the list of displayed hosts in the SDN Controller.

<img width="358" alt="image" src="https://github.com/user-attachments/assets/ec4878c6-19d3-4c6e-8dcb-22efa6fae8d0">
<br/><br/>

Once network traffic is generated, we can connect to the built in graphical interface (DLUX) by entering `192.168.122.122:8181/index.html` into a web browser. refresh the topology with the reload button. It is important to note that whenever a new device is added to the network we should generate some network traffic if needed and refresh the topology with the `Reload` button this is to ensure that the controller and OFM application will see the newly added device.

<img width="592" alt="image" src="https://github.com/user-attachments/assets/cd47c03f-059a-4ece-825c-573b576121b1">
<br/><br/>
