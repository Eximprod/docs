# RJ45 ↔ DB9 serial cable — Cisco IR1101 → inverters <!-- omit from toc -->

> Connecting the **RS‑232** serial interface of the **Cisco IR1101** router to the **RS‑485** serial bus of the inverters or data loggers, through a **Moxa TCC‑80** converter (RS‑232 ↔ RS‑422/485).

- [1. Why this cable is needed](#1-why-this-cable-is-needed)
- [2. The Ethernet cable](#2-the-ethernet-cable)
  - [2.1. Ferrules for stranded cable](#21-ferrules-for-stranded-cable)
  - [2.2. Obtaining the cable](#22-obtaining-the-cable)
- [3. Materials needed](#3-materials-needed)
- [4. Building the cable](#4-building-the-cable)
  - [4.1. Preparing the cable](#41-preparing-the-cable)
  - [4.2. Wiring the DB9 adapter](#42-wiring-the-db9-adapter)
  - [4.3. Configuring the Moxa (DIP)](#43-configuring-the-moxa-dip)
  - [4.4. End-to-end assembly](#44-end-to-end-assembly)
  - [4.5. Connecting to the router](#45-connecting-to-the-router)
  - [4.6. Connecting to the inverter](#46-connecting-to-the-inverter)
- [5. Router configuration (Modbus RTU)](#5-router-configuration-modbus-rtu)
- [6. Signal flow](#6-signal-flow)
- [7. Why the ground (GND) matters](#7-why-the-ground-gnd-matters)
- [8. Equipment and references](#8-equipment-and-references)
- [9. Revisions](#9-revisions)

---

## 1. Why this cable is needed

The serial interface of the **Cisco IR1101** router supports **RS‑232 only**, while the inverters communicate over **RS‑485** using the **Modbus RTU** protocol. To bridge the two, a RS‑232 → RS‑485 conversion is required:

- The router polls the inverters via Modbus RTU; several inverters can be chained on the same RS‑485 bus.
- The router's serial interface (RS‑232, **RJ45** connector) connects to the **RS‑232 (DB9 female)** port of the **Moxa TCC‑80** converter.
- The Moxa converter's **RS‑485** terminal is then wired onward to the inverters' RS‑485 bus (3‑wire link: `+` / `–` / `GND`).
- Between the IR1101 (RJ45) and the Moxa (DB9 female), a **custom cable** is built, with two endings: **RJ45** and **DB9 Male**.

<div align="center">

<img src="images/system_overview.jpg" alt="System overview" width="460" />

*Example installation: the router polls, via Modbus RTU, several inverters chained on RS‑485.*

</div>

<div align="center">

<img src="images/ir1101_front.jpg" alt="Cisco IR1101 - RJ45 serial port" width="264" />

*The Cisco IR1101 router — the RJ45 serial port (RS‑232) the cable plugs into.*

</div>

---

## 2. The Ethernet cable

The serial cables are built **from Ethernet cable** — the wires inside the cable are the ones that carry the signal.

Cable recommendations:

- **Solid‑core (single‑wire) cable.** We recommend solid conductors (one solid wire per conductor), not stranded. Solid wire seats and holds better in the contacts and gives a more stable connection.
- **Shielded cable.** Use **shielded** Ethernet cable. The shield rejects electromagnetic interference and, when properly grounded, drains the noise picked up by the cable.

### 2.1. Ferrules for stranded cable

If you do use **stranded** cable (many thin strands per conductor), you **must** terminate the wire ends with **insulated bootlace ferrules**, properly crimped, **before** inserting them into the terminals.

Stranded wire pushed directly into screw terminals **frays**, makes a poor/intermittent contact, and the strands can touch adjacent terminals. The ferrule gathers all the strands into a **solid tip**, ensuring a firm, safe contact. Choose the ferrule size to match the conductor cross‑section (Ethernet conductors are thin, ~0.2 mm²).

<div align="center">

<img src="images/ferrules.jpg" alt="Insulated bootlace ferrules" width="411" />

*Insulated bootlace ferrules — crimped onto the end of a stranded wire.*

</div>

### 2.2. Obtaining the cable

| Option                        | How to do it                                                                                                                           | Terminations       |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| **A. Both ends made by hand** | Spool of cable (solid‑core, shielded) → cut to length (~100 cm) → terminate **both** ends: RJ45 at one, DB9 (via adapter) at the other | **2** (RJ45 & DB9) |
| **B. Pre‑terminated cable**   | Ready‑made Ethernet cables (factory RJ45) → **cut one end only** → terminate just the **DB9** there                                    | **1** (DB9 only)   |

---

## 3. Materials needed

| Item                                                               | Qty       | Notes                                                                              |
| ------------------------------------------------------------------ | --------- | ---------------------------------------------------------------------------------- |
| **Solid‑core (single‑wire) and shielded** Ethernet cable           | as needed | Spool (opt. A) or pre‑terminated cables (opt. B)                                   |
| Insulated bootlace ferrules                                        | as needed | **Only** if stranded cable is used                                                 |
| D‑Sub 9 **Male** → screw‑terminal block + housing adapter (Delock) | 1 / cable | The DB9 connector at the Moxa side; wires into screws (ferrules for stranded wire) |
| **Moxa TCC‑80** RS‑232 ↔ RS‑485 converter                          | 1         | Has the SW1/SW2/SW3 DIP switches and the R+/D+, R‑/D‑, GND terminals               |

---

## 4. Building the cable

### 4.1. Preparing the cable

At the end that will become the DB9, strip and separate the wires inside the Ethernet cable.

> **Watch the colors:** color codes can vary. If your colors do not match, **go by the PIN number**, not by the color.
> 
<div align="center">

<img src="images/cable_cut.jpg" alt="Ethernet cable cut" width="460" />

*The Ethernet cable: RJ45 at one end, stripped wires at the other.*

</div>

### 4.2. Wiring the DB9 adapter
Use the **Delock "Sub‑D 9 Male → screw‑terminal block"** adapter: each stripped wire is clamped into the screw of its corresponding terminal, per the table below. Identify each wire by its position in the RJ45 layout (color/PIN), then fasten it into the matching DB9 terminal.

<div align="center">

<img src="images/adapter.jpg" alt="Delock D‑Sub 9 Male adapter with screw‑terminal block" width="412" />

*Delock Sub‑D 9 Male adapter with screw‑terminal block and housing: each wire is fastened into the screw of its corresponding pin.*

</div>

**Full RJ‑45 ↔ DB9 correspondence table:**

| RJ‑45 pin | DB9 pin |
| :-------: | :-----: |
|     1     |    6    |
|     2     |    1    |
|     3     |    4    |
|     4     |    5    |
|     5     |    2    |
|     6     |    3    |
|     7     |    8    |
|     8     |    7    |

**Wires actually used** — only 3 pins (2, 3, 5 on the DB9 side), the RS‑232 lines:

| RJ‑45 pin |   →   | DB9 pin | RS‑232 signal    |
| :-------: | :---: | :-----: | ---------------- |
|     4     |   →   |    5    | **GND (ground)** |
|     5     |   →   |    2    | RxD (receive)    |
|     6     |   →   |    3    | TxD (transmit)   |

> **The ground wire (RJ45 4 → DB9 5) must always be connected** (see [section 7](#7-why-the-ground-gnd-matters)).

<div align="center">

<img src="images/rj45_pinout.jpg" alt="RJ45 pin numbering" width="295" />

*RJ‑45 pin numbering (1–8). Go by PIN, not by color.*

</div>

<div align="center">

<img src="images/db9_pinout.jpg" alt="DB9 pinout — Male and Female connector" width="306" />

*DB9 pin numbering — Male and Female connector.*

</div>

**Color codes (for reference only — may differ, verify by PIN):**

|  Pin  | Standard Ethernet (T568B) | Cisco code |
| :---: | ------------------------- | ---------- |
|   1   | white‑orange              | blue       |
|   2   | orange                    | orange     |
|   3   | white‑green               | black      |
|   4   | blue                      | red        |
|   5   | white‑blue                | green      |
|   6   | green                     | yellow     |
|   7   | white‑brown               | brown      |
|   8   | brown                     | white      |

### 4.3. Configuring the Moxa (DIP)
Set **SW1, SW2, SW3** to **ON, ON, OFF**.

<div align="center">

<img src="images/moxa_dip.jpg" alt="Moxa back with DIP switches" width="199" />

*Back of the Moxa TCC‑80 converter: the DIP table and the switches set to ON, ON, OFF.*

</div>

| Mode                            |  SW1   |  SW2   |
| ------------------------------- | :----: | :----: |
| RS‑422 (full‑duplex)            |  OFF   |  OFF   |
| **2‑wire RS‑485 (half‑duplex)** | **ON** | **ON** |
| 4‑wire RS‑485 (full‑duplex)     |   ON   |  OFF   |

| Terminator   |   SW3   |
| ------------ | :-----: |
| Enabled      |   ON    |
| **Disabled** | **OFF** |

→ **ON, ON, OFF** = **2‑wire RS‑485 (half‑duplex)**, with the **terminator disabled**.

### 4.4. End-to-end assembly
Connect in series: **the cable (RJ45)** + **the Sub‑D 9 Male adapter** + **the Moxa converter's DB9 female port**.

### 4.5. Connecting to the router
Plug the **RJ45** connector of the built cable into the **RS (serial) port** of the Cisco IR1101 router.

### 4.6. Connecting to the inverter
The RS‑485 wires from the Moxa converter's terminal block connect to the inverter's RS‑485 terminals.

<div align="center">

<img src="images/moxa_front.jpg" alt="Moxa TCC-80 converter" width="225" />

*The Moxa TCC‑80 converter — the DB9 (RS‑232) port and the RS‑485 terminal block.*

</div>

<div align="center">

<img src="images/moxa_terminal.jpg" alt="Moxa RS-485 terminal block" width="460" />

*Terminal block: T+ , T– , R+/D+ , R–/D– , GND. For 2 wires use R+/D+, R–/D–, GND.*

</div>

| Wire                    |   →   | Moxa terminal |
| ----------------------- | :---: | ------------- |
| The " + " wire          |   →   | **R+ / D+**   |
| The " – " wire          |   →   | **R– / D–**   |
| The "ground/earth" wire |   →   | **GND**       |

> The Moxa terminals are **screw** type → if the wires are stranded, fit **ferrules** on the ends (see [section 2.1](#21-ferrules-for-stranded-cable)).

---

## 5. Router configuration (Modbus RTU)

Besides the cabling, the router's serial port must be configured as a **transparent asynchronous data line**, so that the Modbus application/server can communicate with the inverters through the RS‑232 port (and onward over RS‑485). Apply the following configuration:

```cisco
interface Async0/2/0
 no ip address
 encapsulation relay-line
 async mode dedicated
!
line 0/2/0
 exec-timeout 0 0
 no exec
 transport preferred none
 transport input all
 transport output none
 speed 9600
 databits 8
 parity none
 stopbits 1
!
relay line 0/2/0 0/0/0
!
```

> **Numbering note:** the `0/2/0` and `0/0/0` identifiers depend on the slot and on the serial module installed. Check the available lines with `show line` and adjust them accordingly.

> **Important — serial parameters:** the values `speed 9600`, `databits 8`, `parity none`, `stopbits 1` (i.e. **9600 8N1**) are indicative. These parameters must match **exactly** the inverter's Modbus RTU settings — otherwise communication fails.

---

## 6. Signal flow

```
Cisco IR1101 (RS-232, RJ45)
        │  RJ45 ↔ DB9 Male cable (pins 4→5 GND, 5→2 RxD, 6→3 TxD)
        ▼
Moxa TCC-80 (DB9 female, RS-232)  ──[ SW1=ON, SW2=ON, SW3=OFF ]──►  2-wire RS-485
        │  terminals: R+/D+ , R-/D- , GND
        ▼
RS-485 inverter  ──►  chained inverters (Modbus RTU)
( + → R+/D+ ,  - → R-/D- ,  ground → GND )
```

---

## 7. Why the ground (GND) matters

Connecting the ground wire is **not optional** — on both the RS‑232 side (DB9 pin 5) and the RS‑485 side (the GND terminal). Reasons:

- **RS‑232 is a ground‑referenced (single‑ended) signal.** The receiver decides "1" / "0" by comparing the line voltage **against the common ground**. Without a common reference the levels "float" → corrupted data, CRC errors, an unstable link.
- **Potential differences between grounds.** Without a common GND, the offset between grounds adds onto the signal and can fall outside the valid RS‑232 thresholds → errors. A common ground brings both ends to the same reference.
- **Reducing interference (EMI).** The ground, together with the cable shield, provides a drain path for noise. Without it, interference couples onto the signal → errors, retransmissions. Hence the **shielded cable**.
- **On RS‑485:** although it is differential, **GND** must still be connected to keep the common‑mode voltage within the transceivers' allowed range; otherwise communication can stall or the ports can be damaged.

**Bottom line:** the ground stabilizes the levels, reduces interference and ensures a reliable serial connection.

---

## 8. Equipment and references

- **Cisco IR1101** — Catalyst IR1101 Rugged Series Router
  https://www.cisco.com/c/en/us/products/collateral/routers/1101-industrial-integrated-services-router/datasheet-c78-741709.html
- **Moxa TCC‑80 / TCC‑80I** — serial‑to‑serial RS‑232 ↔ RS‑422/485 converter
  https://www.moxa.com/en/products/industrial-edge-connectivity/serial-converters/serial-to-serial-converters/tcc-80-80i-series)
- **Delock D‑Sub 9 Male → screw‑terminal block adapter** (with housing
  https://conectica.ro/adaptoare-convertoare/bloc-terminal/terminal-block-10-pini-la-serial-d-sub-9-pini-tata-cu-suruburi-carcasa-delock-6
- **Cisco RJ‑45 → DB9 guide**:
  https://www.cisco.com/c/en/us/td/docs/IIOT/routers/ir1101/hw-install-guide/b-ir1101-hig/m-connecting-the-router.html#con_1041724
- **Cisco — serial port configuration**:
  https://developer.cisco.com/docs/iotod/serial/#configure-serial-port-status-for-base-or-expansion-modules-to-cisco-gear

---

## 9. Revisions

| Rev. | Author          | E‑mail                    | Date       |
| :--: | --------------- | ------------------------- | ---------- |
|  1   | Thomas Schaller | thomas.schaller@enedis.fr | 04.06.2025 |
|  2   | Mihnea Marin    | mihnea.marin@epg.ro       | 30.07.2025 |
|  3   | Alexandru Manea | alexandru.manea@epg.ro    | 02.06.2026 |
