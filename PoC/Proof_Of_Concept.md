# ES200 Proof of Concept <!-- omit from toc -->

- [1. Introduction](#1-introduction)
- [2. General Architecture](#2-general-architecture)
  - [2.1. Use cases](#21-use-cases)
  - [2.2. System architecture](#22-system-architecture)
  - [2.3. Supported comunication protocols](#23-supported-comunication-protocols)
- [3. Prerequisites](#3-prerequisites)
  - [3.1 Scope](#31-scope)


# 1. Introduction

ES200 is a control, monitoring and data acquisition solution for the automation of field equipment and processes.

Using modern and secure communication and automation standards, the ES200 is designed to efficiently operate power grids, oil and gas networks, manufacturing units, smart city solutions and so on.

To demonstrate the ES200 solution, a Proof of Concept can be deployed to evaluate the solution's fit to the business and technology requirements.  This document explains the PoC process and its requirements.

# 2. General Architecture

Our solution works in a distributed architecture, along with the software and devices commonly used today. However, ES200 also allows backward compatibility with legacy equipment. Therefore, it can communicate with standard, legacy and modern devices, thus being a versatile solution that greatly improves security and automation functions.

ES200 is able to run, deploy and operate at the Network Edge, while securely isolating SCADA microservices from any other process. It enables data extraction, concentration, processing and storage by acting as a SCADA gateway.

## 2.1. Use cases

ES200 features monitoring, control and communication gateway functions. The system allows the capture of intelligent protection (IED signals), as well as the direct acquisition of digital signals. The ES200 has a wide range of standard protocols (detailed in section 2.4) for monitoring and transmitting information to the higher level. Also, ES200 can store a history of up to 500,000 events stored in non-volatile memory that are described by:

* The time tag identifying the time at which the event occurred (accuracy of 1ms);
* Source of the event;
* Condition before the event;
* Condition after event;

## 2.2. System architecture

The next figures show a general application architecture and an example of a deployment architecture.

<img src="https://raw.githubusercontent.com/Eximprod/docs/main/ES200/images/ES200_System_Architecture.png" width="500" height="300"></p>
Figure 2: The ES200 system architecture

## 2.3. Supported comunication protocols

Communications protocols supported by the ES200 include Modbus, DNP3, IEC 60870-5-104 and IEC 61850. These protocols are currently being used by a wide range of modern protection equipment and IEDs, therefore our solution can be easily deployed with existing equipment. Also, the list of available communication protocols is expanding, other protocols being developed on demand.

Depending on the hardware platform that supports it, the ES200 can directly control and transmit information via its own I/O modules. Below you can find a list of the currently supported communication protocols.


<table>
  <tr>
   <td><strong>Protocol</strong>
   </td>
   <td><strong>Master </strong>
   </td>
   <td><strong>Slave</strong>
   </td>
  </tr>
  <tr>
   <td>Modbus
   </td>
   <td>Yes
   </td>
   <td>Yes
   </td>
  </tr>
  <tr>
   <td>DNP3
   </td>
   <td>Yes
   </td>
   <td>Yes
   </td>
  </tr>
  <tr>
   <td>IEC 60870-5-104
   </td>
   <td>Yes
   </td>
   <td>Yes
   </td>
  </tr>
  <tr>
   <td>IEC 60870-5-101
   </td>
   <td>Yes
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>IEC61850
   </td>
   <td>Yes
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>IEC61850 Edition 2
   </td>
   <td>Yes
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>MQTT
   </td>
   <td>
   </td>
   <td>Yes
   </td>
  </tr>
  <tr>
   <td>LoRa
   </td>
   <td>
   </td>
   <td>Yes
   </td>
  </tr>
  <tr>
   <td>OPC Client
   </td>
   <td>Yes
   </td>
   <td>
   </td>
  </tr>
</table>

Table 2: ES200 supported communication protocols

# 3. Prerequisites

## 3.1 Scope
