# ES200 Proof of Concept <!-- omit from toc -->

[**Table of Contents:**](#toc)
- [1. Introduction](#1-introduction)
- [2. General Architecture](#2-general-architecture)
  - [2.1. Use cases](#21-use-cases)
  - [2.2. System architecture](#22-system-architecture)
  - [2.3. Supported comunication protocols](#23-supported-comunication-protocols)
- [3. Prerequisites](#3-prerequisites)
  - [3.1. Scope](#31-scope)


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

| Protocol              | Master    | Slave |
|-                      |-          |-      |
| Modbus                | Yes       | Yes   |
| DNP3                  | Yes       | Yes   |
| IEC 60870-5-104       | Yes       | Yes   |
| IEC 60870-5-101       | Yes       |       |
| IEC 61850             | Yes       |       |
| IEC 61850 Edition 2   | Yes       |       |
| MQTT                  | Yes       | Yes   |
| LoRa                  |           | Yes   |
| OPC Client            | Yes       |       |

Table 2: ES200 supported communication protocols

# 3. Prerequisites

<table>
  <tr>
   <td><strong>Platform</strong>
   </td>
   <td><strong>Architecture</strong>
   </td>
   <td><strong>vCPU units</strong>
   </td>
   <td><strong>vRAM</strong>
   </td>
   <td><strong>vDisk Space</strong>
   </td>
   <td><strong># of IEDs</strong>
   </td>
   <td><strong>Notes</strong>
   </td>
  </tr>
  <tr>
   <td>Cisco IR1101
   </td>
   <td>ARM 64-bit
   </td>
   <td>200 (17%)
   </td>
   <td>128 MB (15%)
   </td>
   <td>100 MB (10%)
   </td>
   <td>3
   </td>
   <td>Basic Profile <strong><sup>1</sup></strong>
   </td>
  </tr>
  <tr>
   <td>Cisco IR1101
   </td>
   <td>ARM 64-bit
   </td>
   <td>1.155 (100%)
   </td>
   <td>862 MB (100%)
   </td>
   <td>1 GB (100%)
   </td>
   <td>20
   </td>
   <td>Advanced Profile <strong><sup>2</sup></strong>
   </td>
  </tr>
  <tr>
   <td>Cisco IR1835
   </td>
   <td>ARM 64-bit
   </td>
   <td>250 (15%)
   </td>
   <td>256 MB (15%)
   </td>
   <td>200 MB (2,5%)
   </td>
   <td>5
   </td>
   <td>Basic Profile <strong><sup>1</sup></strong>
   </td>
  </tr>
  <tr>
   <td>Cisco IR1835
   </td>
   <td>ARM 64-bit
   </td>
   <td>1.617 (100%)
   </td>
   <td>1.724 MB (100%)
   </td>
   <td>8 GB (100%)
   </td>
   <td>30
   </td>
   <td>Advanced Profile <strong><sup>2</sup></strong>
   </td>
  </tr>
  <tr>
   <td>Cisco IR8340
   </td>
   <td>Intel 64-bit
   </td>
   <td>500 (15%)
   </td>
   <td>256 MB (12 %)
   </td>
   <td>200 MB (10%)
   </td>
   <td>5
   </td>
   <td>Basic Profile <strong><sup>1</sup></strong>
   </td>
  </tr>
  <tr>
   <td>Cisco IR8340
   </td>
   <td>Intel 64-bit
   </td>
   <td>3.465 (100%)
   </td>
   <td>2 GB (100%)
   </td>
   <td>2 GB (100%)
   </td>
   <td>50
   </td>
   <td>Advanced Profile <strong><sup>2</sup></strong>
   </td>
  </tr>
  <tr>
   <td>Virtualized (Kubernetes)
   </td>
   <td>Intel 64-bit
   </td>
   <td>2
   </td>
   <td>1 GB
   </td>
   <td>10 GB
   </td>
   <td>25
   </td>
   <td>Basic Profile <strong><sup>1</sup></strong>
   </td>
  </tr>
  <tr>
   <td>Virtualized (Kubernetes)
   </td>
   <td>Intel 64-bit
   </td>
   <td>8
   </td>
   <td>32 GB
   </td>
   <td>100 GB
   </td>
   <td>200
   </td>
   <td>Advanced Profile <strong><sup>2</sup></strong>
   </td>
  </tr>
</table>

<sup><b>1</b></sup> Min use of Edge Computing Resources - ES200 uses ~15% of available resources on this profile

<sup><b>2</b></sup> Max use of Edge Computing Resources - suitable for more demanding edge deployment


## 3.1. Scope
