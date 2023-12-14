# ES200 Installation Guide

# 1. About this manual
The ES200 software is designed to solve a large variety of real-life automation challenges. By using the communication standards of today’s industries, ES200 is a modern and scalable solution that can be easily integrated with most of the existing automation systems on the market. This document explains the procedure needed to install and run the ES200 software on a Cisco IOx enabled router.

## 1.1. Legal Information
The information in this document is subject to change without prior notice and is not a commitment on the part of the supplier. Eximprod does not take responsibility for the use of the information in this document.

Eximprod is not responsible for any direct or indirect matter of any nature that may result from the use of this document or any product mentioned in the document.

The information in this document cannot be reproduced or copied without the written permission of Eximprod and the content cannot be given to a third party for unauthorized use.

## 1.2. General dispositions
This document provides information about the ES200 software and its installation process. The information available in this manual is intended for personnel who will use this product to configure different components or for current use.

## 1.3. Abbreviations and terminology

<table>
  <tr>
   <td>
    Abbreviations
   </td>
   <td>
    Description
   </td>
  </tr>
  <tr>
   <td>
    RTU
   </td>
   <td>
    Remote Terminal Unit
   </td>
  </tr>
  <tr>
   <td>
    IED
   </td>
   <td>
    Intelligent Electronic Devices
   </td>
  </tr>
  <tr>
   <td>
    DC
   </td>
   <td>
    Direct current
   </td>
  </tr>
  <tr>
   <td>
    SSH
   </td>
   <td>
    Secure Shell
   </td>
  </tr>
  <tr>
   <td>
    IOS
   </td>
   <td>
    Cisco operating system
   </td>
  </tr>
  <tr>
   <td>
    IOx
   </td>
   <td>
    <strong>IO</strong>S + Linu<strong>x </strong>= IOx
   </td>
  </tr>
  <tr>
   <td>
    DHCP
   </td>
   <td>
    Dynamic Host Configuration Protocol
   </td>
  </tr>
  <tr>
   <td>
    S0 / S1
   </td>
   <td>
    Cisco IR809 serial ports
   </td>
  </tr>
  <tr>
   <td>
    VM
   </td>
   <td>
    Virtual Machine
   </td>
  </tr>
  <tr>
   <td>
    TCP
   </td>
   <td>
    Transmission Control Protocol
   </td>
  </tr>
</table>

# 2. Introduction
ES200 is a control, monitoring and data acquisition unit dedicated to applications in the industry field (utilities, energy, gas etc.). It is the ideal solution both for automation and control of transformation, power or connection points and for local SCADA systems.

ES200 has been developed in line with the latest international standards. Our engineering teams have used their expertise in the field to develop a solution that is able to function in a distributed architecture along with most of the solutions available at present time.

ES200 features monitoring, control and communication gateway functions. The system allows the capture of intelligent protection (IED signals), as well as the direct acquisition of digital signals. It has a wide range of standard protocols (Modbus TCP/RTU, DNP3 TCP/RTU, IEC 60870-5-101/104, IEC 61850, LoRaWAN) for monitoring and transmitting information to the higher level.
## 2.1. System Architecture
You can install the ES200 software on the Cisco routers in the following ways:

* Local Manager
* ioxclient
* Fog Director
* Field Network Director

This manual will cover the installation of the ES200 software using the Local Manager of the IOx.

## 2.2. Hardware
### 2.2.1. IR809
The IR809 is Cisco’s smallest multimode 3G and 4G LTE wireless router, making it an excellent solution for distribution automation and remote asset management across multiple industrial vertical markets (Figure 1). The IR809 has an integrated 9.6 to 60V DC power input and is designed to withstand hostile environments, including shock, vibration, dust, and humidity, and supports a wide temperature range (–40 to 60°C and type-tested at 85°C for 16 hours). The IR809 brings together:
* Enterprise-class wireline-like services, such as quality of service (QoS)
* Cisco advanced virtual private network (VPN) technologies (such as Dynamic Multipoint VPN [DMVPN] and Flexible VPN [FlexVPN])
* Multiple Virtual Routing and Forwarding (VRF) instances for cellular highly secure data, voice, and video communications
* Cisco IOx, an open, extensible environment for hosting applications at the network edge

The IR809 also extends connectivity to include low-power wide-area (LPWA) access using the Cisco Interface Module for LoRaWAN.

### 2.2.2. IR1101

Cisco IR1101 delivers secure IoT connectivity for today and the future. Its 5G ready modular design allows you to upgrade to new communications protocols when they become available, avoiding costly rip-and-replace. Add or upgrade WAN, edge compute and storage components as technologies and your needs evolve. With its rugged hardware and compact form-factor, you can install it almost anywhere.

## 2.3. Software
### 2.3.1. ES200
ES200 is a Software-Defined Remote Terminal Unit (RTU), operating on Cisco hardware running the IOx application hosting platform (e.g. IR809, IR1101) that enables multiple use cases for utilities and:
* offers the ability to manage large scale IoT sensor data
* operates and controls the distributed network infrastructure
* allows the update and reconfiguration of the software functions
* shares information with other third-party applications that run on the Cisco hardware
* improves security and automation functionalities while allowing backward compatibility with legacy equipment

# 3. IOS configuration
In order to run an IOx application on Cisco routers one must do a router configuration before installing the application. In the next chapters, this manual will provide information on how to configure Cisco IR809 and Cisco IR1101 in order to have the IOx platform enabled for applications.

This manual will assume that the network configurations of the router are already done and SSH is enabled on the device. Also, this guide will assume that GE0 is configured as a WAN port and GE1 is configured as a LAN port.

Because ES200 is an application that requires connections from the WAN, the access to the ES200 container can be done in two different ways:

* Using a port forwarding from the WAN interface to the ES200 container
* Using NAT with another IP from the WAN network to the IP of the ES200 container (IR1101) or the IOx VM (IR809)

This manual will explain the second method.

## 3.1. IR809
The Cisco IR809 is an industrial router that runs a LynxWorks hypervisor on top of which runs three Virtual Machines (VM), the IOS VM, the IOx VM and the Virtual Device Server. To find out more about how this works please visit this [link.](https://developer.cisco.com/docs/iox/%23!ir-800-series-platform-information/ir8xx-platforms)

Our VMs of interest are IOS and IOx. IOS VM is the virtual machine which executes the routing functions, while the IOx VM is the one which executes the Cisco Application Framework (CAF) on which ES200 software runs. The IOx VM is named GuestOS in IOS.

Between those two VMs there is an emulated Ethernet link. This links supports IPv4 and IPv6. The IOS end of the emulated link is managed by GE2 virtual interface on IR809 and the IOx end is managed using a DHCP server.

The configuration of the IR809 IOS has three steps that must be accomplished for the ES200 software to run properly:

* Configuration of the network interface 
* Configuration of a DHCP server for IOx VM 
* Configuration of the serial interfaces (optional)

For detailed information on what and how you can configure furthermore the GuestOS in the Cisco IR809 router, please follow this [link.](https://www.cisco.com/c/en/us/td/docs/routers/access/800/829/software/configuration/guide/b_IR800config/b_guestos.html)

### 3.1.1. Network interfaces configuration

To have connectivity between the IOS and IOx one must configure the GE2 interface on the router. Our recommendation is to use a /30 network as you will need the same IP address of the IOx VM every time. In this example, the GE2 interface’s IP address will be set to 192.168.2.1/30.

Also, to ensure connectivity between them, the IOx needs IPv6 enabled on the interface with autoconfiguration on.

```
R1-epg(config)# interface GigabitEthernet 2
R1-epg(config-if)# ip address 192.168.2.1 255.255.255.252
R1-epg(config-if)# ip nat inside
R1-epg(config-if)# ipv6 address autoconfig
R1-epg(config-if)# ipv6 enable
R1-epg(config-if)# no shutdown
```

As one can see, we activated the *ip nat inside* which will allow traffic from the WAN to arrive to the IOx using NAT. In order to achieve this, one must enable *ip nat outside* on the WAN interface, which in our case is GE0.

Once we enabled this, one can also configure the NAT rule, In our case the NAT is done IP to IP as follows:
```
R1-epg(config)# ip nat inside source static 192.168.2.2 10.10.30.182	
```

This allows the WAN packets to access the IOx VM using the IP 10.10.30.182.

### 3.1.2. DHCP server for IOx VM

Now that one has configured the IOS networking part, we need to make sure that the IOx VM will have access to a DHCP server from where it can take an IP address. For that, it is necessary to configure a DHCP server on the IOS. The DHCP server will be for the network 192.168.2.0/30 and will have a the default router set on 192.168.2.1.

```
R1-epg(config)# ip dhcp pool gospool
R1-epg(dhcp-config)# network 192.168.2.0 255.255.255.252
R1-epg(dhcp-config)# default-router 192.168.2.1
R1-epg(dhcp-config)# domain-name epg.ro
R1-epg(dhcp-config)# dns-server 8.8.8.8
R1-epg(dhcp-config)# exit
```

### 3.1.3. Serial interfaces configuration (optional)

IR809 has two serial ports, S0 and S1. The S0 serial port can act as a RS232 or RS485 interface, and the S1 serial port can only act as an RS232 interface.

To relay the information coming from the S0 port to the ES200 application one must do the following IOS configuration:

```
R1-epg(config)# interface Async0
R1-epg(config-if)# encapsulation relay-line
R1-epg(config-if)# exit
R1-epg(config)# line 1
R1-epg(config-line)# transport input all
R1-epg(config-line)# exit
R1-epg(config)# relay line 1 1/5 propagation

```

To relay the information coming from the S1 port to the ES200 application one must do the following IOS configuration:

```
R1-epg(config)# interface Async1
R1-epg(config-if)# encapsulation relay-line
R1-epg(config-if)# exit
R1-epg(config)# line 2
R1-epg(config-line)# transport input all
R1-epg(config-line)# exit
R1-epg(config)# relay line 2 1/6 propagation
```

## 3.2. IR1101

The Cisco IR1101 is an industrial router that runs IOS-XE, an open and flexible operating system optimized for the new era of enterprise networks. In here, the Cisco Application Framework (CAF) is run as a process inside the IOS-XE.

In order to run the ES200 application on the Cisco IR1101, the following steps must be done:

* Enable iox
* Configure the networking
* Configure NAT
* Configuration of the Serial interface (optional)

For further more information about what other things can be configured on the IR1101 IOx part, please visit the following [link.](https://www.cisco.com/c/en/us/td/docs/routers/access/1101/software/configuration/guide/b_IR1101config/b_IR1101config_chapter_010001.html)

### 3.2.1. Enabling IOx

To enable iox, one must have the http/https server configured and also a username and a password set in the IOS. To do so, use the following commands:

### 3.2.2. Network configuration

The network architecture between the IOS-XE and the CAF is different than the one in the IR809. In here, the IOS-XE hosts a bridge network called VirtualPortGroup0.

To configure this interface, one must follow the following commands:

```
R2-epg(config)# interface virtualportgroup 0
R2-epg(config-if)# ip address 192.168.2.1 255.255.255.0
R2-epg(config-if)# ip nat inside
```

### 3.2.3. NAT configuration

As discussed also for the IR809, for the WAN connections to arrive into the ES200 application, we will configure a NAT. We need to enable _ip nat outside _on the GigabitEthernet 0/0/0 interface (we assume that this is the WAN interface). After doing so, one must do a NAT rule between the WAN IP address and the ES200 application IP address. We also assume that the IP address of the ES200 application will be 192.168.2.2 and the IP address for the WAN will be 10.10.30.183.

To do so, one must follow the following commands:

```
R2-epg(config)# interface GigabitEthernet 0/0/0
R2-epg(config-if)# ip nat outside
R2-epg(config-if)# exit
R2-epg(config)# ip nat inside source static 192.168.2.2 10.10.30.183
```

### 3.2.4. Serial interface configuration (optional)

The IR1101 has an RS232 serial port that can be relayed into any IOx application on the router. To do this, one must do the following IOS configuration:

```
R1-epg(config)# interface Async0/2/0
R1-epg(config-if)# encapsulation relay-line
R1-epg(config-if)# exit
R2-epg(config)# relay line 0/2/0 0/0/0

```

# 4. ES200 Software installation

The ES200 software is wrapped up in a Docker container which is then converted into an LXC container using the _ioxclient _application provided by Cisco. The latest version of the ES200 application is 20.01.02LTS. Please make sure that when you install the software on the router you have the same version as stated in this document. If you do not have the latest version, please contact [es200_support@epg.ro.](mailto:es200_support@epg.ro)

## 4.2. IR809

First step is to open a browser and acces https://IP:8443, where IP is the one used on NAT configuration, in this example 10.10.30.182.

Use a username and a password that you have set as an administrator on the router to login.

Check in the System settings tab that Application Signature is disabled.

In order to install ES200, go to Applications tab then press add new.

In the window shown, fields must be completed with: application ID is ES200 and Application Archive.

Now that we added the ES200 application, it must appear in the Applications tab with DEPLOYED status. To activate ES200, press Activate.

In the shown menu chose async0 and iox-nat0, then press Port Mapping.

In the shown menu chose Custom then press OK. After you're done press Activate from Activate application menu.

Now ES200’s status must be ACTIVATED. To start the application, from Application tab press Start.

After pressing Start, application status will change to RUNNING from ACTIVATED.

## 4.3. IR1101

For the IR1101, the procedure to install the ES200 is mostly the same. The difference between the two procedures are that the IOx web interface is accessible from the IOS-XE web interface and the networking of the application is part of the VirtualPortGroup0.

First step is to open a web browser and navigate to the https://[IP]. This will send you to the login page.

After you login, use the left menu to go to the IOx web interface

From this step you need to do the same things as in the IR809 installation process. Check in the System settings tab that Application Signature is disabled.

In order to install ES200, go to Applications tab then press add new.

In the window shown, fields must be completed with application ID is ES200 and Application Archive.

Now that we added the ES200 application, it must appear in the Applications tab with DEPLOYED status. To activate ES200, press Activate.

In the shown menu chose async0 and VPG0, then press Interface settings.

In the shown menu chose Static and configure the IP addresses as shown. If you didn’t use this manual to configure the VPG0 interface, then the IP address in this menu must correspond to an assignable IP address from the VPG0 interface network. After completing to fill in the form, press OK. Then press Activate.

Now ES200’s status must be ACTIVATED. To start the application, from Application tab press Start.

After pressing Start, application status will change to RUNNING from ACTIVATED.