# SG200 — Pre-Engagement Questionnaire

**Purpose:** This questionnaire gives us the minimum information needed to size an SG200 deployment (Essential or Professional), choose the right architecture (standalone per site vs. central platform + site kits), and come back with a realistic budgetary offer.

**How to answer:**

- Fill in what you know. Rough answers are fine — an estimate is more useful than a blank.
- Use the legend below instead of leaving fields empty, so we can tell *"no"* apart from *"we don't know yet"*.
- Section A is one row per site. Sections B–H apply to the whole organization; note per-site exceptions where relevant.

**Legend:** `Y` = yes / in place · `N` = no / not present · `P` = partial · `U` = unknown, to be confirmed · `N/A` = not applicable

---

## 0. Contacts

| Field | Answer |
|---|---|
| Company / group name | |
| Main contact (name, role) | |
| Email / phone | |
| Single point of contact for cybersecurity | |

---

## A. Site inventory

### A1. Sites, assets and control systems (one row per site)

| # | Site name | Operating entity (legal name) | Type (solar / wind / BESS / substation / hydro) | Installed power [MW] | RTU / PPC vendor & model | Protocols in use | OT device count (approx.) | Northbound connections (count) |
|---|---|---|---|---|---|---|---|---|
| 1 | | | | | | | | |
| 2 | | | | | | | | |
| 3 | | | | | | | | |

> **Protocols:** IEC 60870-5-104, IEC 60870-5-101, IEC 61850 (MMS / GOOSE), DNP3, Modbus TCP / RTU, OPC UA, PROFINET, EtherNet/IP, proprietary.
> **OT devices:** RTUs, PLCs, gateways, IEDs / protection relays, power quality meters, weather stations, inverter / turbine controllers, BMS / PCS / EMS controllers.

### A2. Network and connectivity (one row per site)

| # | Site name | Topology (flat / VLANs) | Redundancy (RSTP / MRP / PRP / HSR / none) | Existing network equipment (vendor & model: router, switch, firewall) | WAN connection (fibre / LTE / 5G / satellite) | Bandwidth (down / up) | Remote access method today |
|---|---|---|---|---|---|---|---|
| 1 | | | | | | | |
| 2 | | | | | | | |
| 3 | | | | | | | |

---

## B. Control systems (OT / SCADA)

1. Which SCADA / park power controller (PPC) platforms are in use, and from which vendors? Note per-site differences.
2. Are there local HMI or SCADA servers on site? If yes: operating system and rough patch status.
3. Who are the northbound counterparties (DSO, TSO, dispatch centre, trader / aggregator), and who manages each of those links — you, the counterparty, or a third party?
4. Which inverter / PCS / BMS vendors are deployed, and do any of them have a cloud or vendor-portal connection to the sites (vendor monitoring, firmware updates)?

## C. Network & addressing

1. Do sites use overlapping private IP ranges, or is addressing unique per site? *(drives the VPN hub design)*
2. Do the WAN links have static public IPs, or are the mobile links behind carrier-grade NAT (CGNAT)? *(drives the VPN architecture)*
3. Any monthly data caps or throughput limits on mobile links?
4. Do the sites have managed switches with port mirroring (SPAN), or spare ports where a monitoring sensor could be connected? *(needed for passive OT monitoring)*
5. What network documentation exists per site?

   - [ ] Network diagram
   - [ ] IP address plan
   - [ ] Asset / device list
   - [ ] None

## D. Remote access & existing IT / security stack

1. How are the sites reached remotely today (VPN, vendor remote access, jump host)? Who holds the accounts?
2. Which third parties currently have remote access (O&M contractor, inverter vendor, SCADA integrator, DSO)? List them.
3. Which central systems already exist, in-house or via an MSSP?

   - [ ] Backup solution
   - [ ] Identity / Active Directory / LDAP
   - [ ] Multi-factor authentication (MFA)
   - [ ] Monitoring / SIEM
   - [ ] Ticketing / ITSM
   - [ ] None of the above

4. Is there an existing central location (HQ, dispatch centre, datacentre, cloud tenancy) suitable to host the central SG200 node?

## E. On-site environment (note per-site exceptions)

1. Spare rack space (how many U) or DIN-rail space in the control cabinet?
2. Power available for new equipment: 230 V AC or 48 V DC? UPS-backed? Spare breaker positions?
3. Environment: climate-controlled room or outdoor cabinet (temperature, dust)?

## F. Security & documentation status

1. Any prior OT risk assessment, penetration test, or security audit? When, and by whom?
2. Does an asset inventory exist? In what form (spreadsheet, tool), and how current is it?
3. Which security documentation is in place?

   - [ ] Information / OT security policy
   - [ ] Incident response plan
   - [ ] Business continuity / disaster recovery plan
   - [ ] Backup & restore policy
   - [ ] Access management policy
   - [ ] None of the above

4. Are logs currently collected anywhere, and for how long are they retained?
5. Is there an incident reporting process today — who gets notified, and within what timeframe?

## G. NIS2 / compliance

1. List all operating entities (SPVs) in scope and, for each: standalone or part of a larger group? *(this affects the essential / important classification)*
2. Per entity: have you self-assessed or registered with the national authority (e.g. DNSC in Romania)?
3. Classification received or expected per entity: essential / important / unknown.
4. Who owns cybersecurity on your side: IT, OT / SCADA engineering, a CISO, or an external provider?
5. Any other compliance drivers: ISO 27001, IEC 62443, energy-regulator requirements, insurance, customer or audit requirements?

## H. Practicalities & planning

1. Any zero-downtime constraints or maintenance windows we need to respect? Can planned curtailment windows be used for installation work?
2. Who runs the sites day to day — your own team or an outsourced O&M provider? Can they provide hands-on-site support during installation?
3. Preferred deployment model, if you already have a view: standalone kit per site, or a central platform with lightweight site kits? *(both are supported)*
4. How many sites are under construction or in the pipeline for the next 24 months, and of what type? *(for sizing the central platform)*
5. Rough timeline for the project, and a ballpark budget if you have one.

---

None of this needs to be precise. Even rough answers are enough for us to tailor the design and come back with a realistic number.
