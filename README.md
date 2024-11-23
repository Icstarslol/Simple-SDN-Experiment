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

## Configuring the Open vSwitch Management
For this experiement we will be using the 
