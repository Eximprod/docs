# Ghid de teren — integrarea ES200 la prosumatori LV PV - DEER VPP <!-- omit from toc -->

---

- [1. Scop, filozofie și regula de aur](#1-scop-filozofie-și-regula-de-aur)
  - [1.1. Cele trei principii care guvernează vizita](#11-cele-trei-principii-care-guvernează-vizita)
- [2. Ce duci cu tine (kit de teren).](#2-ce-duci-cu-tine-kit-de-teren)
- [3. Mașina de stări (overview)](#3-mașina-de-stări-overview)
- [4. S0 — Survey și reconciliere cu dosarul](#4-s0--survey-și-reconciliere-cu-dosarul)
  - [4.1. Cum citești topologia reală (primele 30 de minute)](#41-cum-citești-topologia-reală-primele-30-de-minute)
- [5. S1 — Connection point: la ce ne conectăm](#5-s1--connection-point-la-ce-ne-conectăm)
  - [5.1. Există logger → ne conectăm LA logger. Nu facem bypass la invertoare.](#51-există-logger--ne-conectăm-la-logger-nu-facem-bypass-la-invertoare)
  - [5.2. Nu există logger → ne conectăm la invertoare.](#52-nu-există-logger--ne-conectăm-la-invertoare)
- [6. S1.5 — Coexistență cu cloud-ul + arbitrarea controlului](#6-s15--coexistență-cu-cloud-ul--arbitrarea-controlului)
- [7. S2 — Medium: cum ne conectăm fizic](#7-s2--medium-cum-ne-conectăm-fizic)
  - [7.1. S2.detail — RS485 multidrop la N invertoare (fără logger)](#71-s2detail--rs485-multidrop-la-n-invertoare-fără-logger)
- [8. S3 — Acces local și activarea canalului Modbus/IEC104](#8-s3--acces-local-și-activarea-canalului-modbusiec104)
- [9. S3.5 — Mai multe echipamente: agregare și distribuție](#9-s35--mai-multe-echipamente-agregare-și-distribuție)
- [10. S4 — Validare: întâi direct prin ES200](#10-s4--validare-întâi-direct-prin-es200)
- [11. S5 — Read + Write în siguranță](#11-s5--read--write-în-siguranță)
- [12. S6 — Northbound IEC104 (în mare, remote)](#12-s6--northbound-iec104-în-mare-remote)
- [13. S7 — Montaj și alimentare](#13-s7--montaj-și-alimentare)
- [14. Information harvest + as-built](#14-information-harvest--as-built)

---

## 1. Scop, filozofie și regula de aur

Configurația funcțională a ES200 este simplă și fixă, **indiferent de site**:

- **Southbound:** Modbus TCP sau Modbus RTU către logger / invertor / master inverter.
- **2 măsuri RO:** P și Q (la mai multe echipamente → agregate în **Multi Data Master** ca `P_SUMMED`, `Q_SUMMED`).
- **2 setpoint-uri RW:** `SET_P`, `SET_Q` (distribuite southbound către fiecare echipament).
- **Northbound:** IEC104 cu 4 puncte (2 RO + 2 RW) către VPP.

**Problema grea nu e ES200. E terenul.** Echipamentul real diferă de documentație, topologia de comunicație e neclară, lipsesc parole/adrese/firmware, nu se știe dacă setpoint-urile sunt suportate și unde se scriu, iar uneori nu există nici spațiu/alimentare pentru router.

### 1.1. Cele trei principii care guvernează vizita

1. **Vizita nu se anulează niciodată.** Mereu încercăm să integrăm ce găsim fizic în teren. Nu plecăm acasă cu mâna goală.
2. **Lasă-l cablat.** Dacă nu poți termina configurarea logică, termină **stratul fizic** (montaj + cablare + alimentare + rețea), ca restul să se poată face **remote**.
3. **Culege tot (information harvest).** Mai ales când terenul ≠ documentația: poze, screenshot-uri din web UI, diagnostice Modbus/IEC104. Orice îți permite să închizi site-ul remote sau să faci următoarea vizită aproape sigură.

> **Regula de aur:** nu pleca la drum pe baza unei singure surse. Documentația e punct de plecare, **nu adevăr de teren**

---

## 2. Ce duci cu tine (kit de teren).

**Hardware**
- IR1101 cu ES200 **preprovisionat** (config făcut de acasă pentru echipamentul așteptat).
- Cablu serial RJ45↔DB9 + **Moxa TCC-80** (RS232→RS485) — IR1101 are doar RS232 (vezi ghidul de cablare serială).
- **Adaptor USB→DB9 (RS232)** pentru laptop + **driver instalat** (atenție la COM-port în Device Manager) + Moxa de rezervă pentru RS485.
- **Șină DIN + suport/bracket** de rezervă (multe dulapuri nu au loc pregătit).
- **Sursă DIN 230VAC→24/48VDC** (IR1101 cere 24–48V DC; multe site-uri au doar 230V AC sau UPS).
- Cabluri Ethernet (solid-core, ecranat), papuci/ferule, switch mic de rezervă.

**Software / acces**
- Aplicații de commissioning per vendor: **FusionSolar** (Huawei), **SolaX installer / Solarman**, **Deye/Solarman**, **Solis SolisCloud / installer**, **Fronius Solar.web / Datamanager**, etc.
- **Modbus Poll** (principal) și/sau **QModMaster** pentru diagnostic southbound.
- VM cu **Triangle Protocol Test Harness** pentru diagnostice detaliate.
- **Biblioteca offline de register maps**. Asta îți permite să integrezi un vendor neașteptat fără a doua deplasare.
- Credențialele site-ului: de multe ori sunt **conturi de cloud**, nu acces local pe echipament.

---

## 3. Mașina de stări (overview)

**Firul principal:** `S0 → S1 → S1.5 → S2 → S3 → S3.5 → S4 → S5 → S6 → S7 → Harvest → Exit`. Fiecare stare are o **decizie** și **ramuri**; mai jos e rezumatul, detaliile complete sunt în secțiunile următoare.

**S0 · SURVEY** (§4) — identifică fizic *fiecare* echipament; reconciliază cu dosarul (presupune că dosarul minte).
- Decizie: există **data-logger**, sau doar **invertoare**?
- ⮑ teren ≠ dosar → **NU pleci**. Ai register map-ul real? reconfig ES200 pe loc; dacă nu → *harvest* + *lasă cablat* + raport.

**S1 · CONNECTION POINT** (§5) — la ce ne legăm?
- ⮑ **există logger** (și poate fi *slave*) → te legi **LA logger**, fără bypass la invertoare.
- ⮑ **fără logger** → la invertoare: *master* prezent → la master; *standalone* → configurezi fiecare.

**S1.5 · COEXISTENCE** (§6) — loggerul are uplink cloud (FusionSolar/SolaX…)? nu-l strica.
- Activează un canal **Modbus/IEC104-slave separat**; ES200 trebuie să câștige arbitrarea.
- ⛔ **NO-GO:** loggerul nu acceptă al 2-lea canal **sau** cloud-ul suprascrie setpoint-ul.

**S2 · MEDIUM** (§7) — cum ne legăm fizic?
- ⮑ **invertor** → Modbus **RTU** întâi (fără IP/config extra pe router).
- ⮑ **logger** → de regulă Modbus **TCP** (mai stabil, dar cere reconfig router + ES200).
- RS485 ⇒ **Moxa** (IR1101 = doar RS232). N invertoare fără logger → un bus multidrop + **reasignează ID-uri** (§7.1).

**S3 · ACCESS + ENABLE** (§8) — acces **LOCAL** pe echipament (nu doar cloud); activează Modbus; citește parametrii reali.
- Preferință: **web UI (TCP)** > app pe USB/WiFi > ecran fizic. ⛔ **Blocat → escaladezi.**

**S3.5 · MULTI-DEVICE** (§9) — N echipamente: **Multi Data Master** → `P_SUMMED`/`Q_SUMMED`; setpoint **distribuit la toate** (procent preferat → același % la toate; absolut → proporțional cu puterea nominală).

**S4 · VALIDATE** (§10) — testează **direct prin ES200**.
- ⮑ **merge?** → gata, pleci. ⮑ **probleme?** → laptop (Modbus Poll / Triangle) ca să izolezi *teren* vs *config*.

**S5 · READ + WRITE** (§11) — citești P/Q reale; scrii **setpoint de test pe UN echipament**, cu operatorul de față; verifici **feedback real** (nu doar „accepted"); testezi recovery; nu strici cloud-ul.
- Q se testează live **doar** dacă DEER îl folosește și e sigur.

**S6 · NORTHBOUND** (§12) — în mare **remote**. În teren confirmi doar: **SIM APN sus** + networking ajunge la IR1101+ES200 prin VPN.

**S7 · MOUNT + POWER** (§13) — montaj pe **DIN**, alimentare **24–48V DC** (sau sursă 230VAC→DC din kit). Mereu: **lasă cablat**.

**HARVEST + AS-BUILT** (§14) — poze, screenshot-uri, register map confirmat, slave/unit IDs, IP-uri, limitări cunoscute.

**Exit:** `INTEGRAT` · `REMOTELY-COMPLETABLE` · `BLOCAT-DOCUMENTAT` — niciodată „abandonat".

> **Failsafe:** la orice cădere, echipamentele rămân la **ultimul setpoint** (hold-last, pe registre *stored*). ES200 nu revine la default.

---

## 4. S0 — Survey și reconciliere cu dosarul

**Acțiune:** identifică fizic FIECARE echipament — etichete, ecrane, dulap, porturi, cablaje. Notează model exact, putere nominală, număr de echipamente.

**Întrebarea de decizie:** există **data logger** (SmartLogger, DataHub1000, SDongle, WebdynSun, dongle vendor) sau doar **invertoare**?

**Dacă terenul ≠ documentația**:

1. **NU pleci.** Identifică echipamentul real.
2. Dacă ai **register map-ul** vendorului real în bibliotecă + adaptoarele + acces → reconfigurezi ES200 pe loc și integrezi echipamentul **real**.
3. Dacă mismatch-ul te lasă fără register map / cablare / credențiale → **harvest masiv** (poze, screenshot, diagnostic) + **lasă cablat** stratul fizic + raport exact cu ce lipsește. Restul se face remote sau la următoarea vizită.

### 4.1. Cum citești topologia reală (primele 30 de minute)

Tot ce urmează (S1 connection point, S2 medium, S3.5 număr de echipamente) depinde de a răspunde corect la: **există logger? un invertor e master pentru restul? sunt înlănțuite sau standalone?** Dosarul minte, iar operatorul de multe ori nu știe.

1. **Traseu fizic întâi:** urmărește cablurile — găsește **fiecare** invertor, urmărește fiecare run RS485/Ethernet ca să identifici un logger sau un master și porturile libere; fotografiază **fiecare etichetă**.
2. **Confirmare electrică:** verifică ecranele, apoi scanează bus-ul/rețeaua ca să vezi ce răspunde.
3. **Documentația echipamentului, live:** verifică documentația echipamentelor găsite — din biblioteca ta offline, de pe net, sau cu un LLM — ca să afli cum comunică, ce porturi au și ce porturi sunt disponibile. Esențial când dai de un echipament neașteptat.

---

## 5. S1 — Connection point: la ce ne conectăm

### 5.1. Există logger → ne conectăm LA logger. Nu facem bypass la invertoare.

Motivul: invertoarele sunt deja înlănțuite (RS485) la logger, care le agregă și le poate controla. Loggerul e de regulă deja integrat de investitor într-o platformă cloud (FusionSolar / SolaX Cloud) pentru monitorizare și control. **Nu vrem să stricăm asta.** Ne legăm de logger și activăm pe el un canal **Modbus-slave / IEC104-slave separat**, fără să afectăm uplink-ul cloud.

> **Atenție — nu orice logger poate fi slave.** Loggerul e connection point valid pentru control doar dacă poate acționa ca **Modbus/IEC104 slave** interogabil de ES200. Unele loggere (ex. **WebdynSun**) sunt **master-only** și **nu pot fi interogate** → atunci connection point-ul devine invertorul direct. Confirmă întâi că loggerul expune un slave.

### 5.2. Nu există logger → ne conectăm la invertoare.

- **Master inverter:** dacă un invertor e master RS485 pentru ceilalți → te conectezi la master (ca la un mini-logger).
- **Standalone:** dacă invertoarele lucrează individual (uneori nici nu erau cablate să comunice) → trebuie configurat **fiecare** (activezi Modbus, setezi **unit ID unic**), eventual le înlănțui tu pe un bus RS485 comun.

---

## 6. S1.5 — Coexistență cu cloud-ul + arbitrarea controlului

Loggerul are deja un **uplink cloud**. Două riscuri:

1. **Al doilea canal:** loggerul poate să nu accepte o a doua sesiune Modbus master/un al doilea canal, sau activarea Modbus local poate perturba sesiunea cloud.
2. **Arbitrarea controlului:** dacă și cloud-ul, și ES200 pot scrie setpoint-uri — **cine câștigă?**

**Politica:** activează un canal local **separat** de uplink; verifică explicit că **sesiunea cloud supraviețuiește**; confirmă că prioritatea sursei de control (control source) face ca **setpoint-ul ES200 să suprascrie cloud-ul**.

**NO-GO / escaladare:** loggerul **refuză** al doilea canal.

---

## 7. S2 — Medium: cum ne conectăm fizic

Mediul nu e liber, e dictat de connection point:

| Connection point      | Medium preferat                                    | De ce                                                                         |
| --------------------- | -------------------------------------------------- | ----------------------------------------------------------------------------- |
| **Invertor (direct)** | **Modbus RTU** (RS232 direct, sau RS485 prin Moxa) | Nu necesită IP-uri și config extra pe IR1101 — merge cu ES200 preprovisionat. |
| **Logger**            | **Modbus TCP**                                     | Loggerele rar comunică serial northbound; de regulă doar TCP/IP.              |

**Compromisul cheie:**
- **Serial (RTU):** simplu, fără reconfig — config-ul preprovisionat funcționează ca atare. Mai puțin stabil.
- **TCP:** mai stabil și robust, **dar** cere modificări **în teren** la config-ul de rețea al routerului **și** la ES200-ul preprovisionat (IP, subnet, mască, gateway). Pentru TCP: ori IP-ul **default** al echipamentului, ori întrebi investitorul care e rețeaua lui.

> **Hardware:** IR1101 are serial **doar RS232**. Orice RS485 trece prin
> **Moxa TCC-80** (SW1=ON, SW2=ON, SW3=OFF → RS485 2-wire). Vezi ghidul
> de cablare seriala.

### 7.1. S2.detail — RS485 multidrop la N invertoare (fără logger)

IR1101 are **un singur** port serial → toate invertoarele stau pe **un singur bus RS485 multidrop**:

- **Reasignează adresele Modbus** pe fiecare invertor înainte de înlănțuire — invertoarele care n-au fost niciodată în rețea vin de regulă cu **aceeași adresă default (ex. 1)** → coliziune. Setezi o adresă unică pe fiecare (din ecran/app).
- **Același baud / 8N1** pe toate echipamentele de pe bus (trebuie să fie identice cu config-ul ES200/IR1101).
- **Terminație la cele două capete fizice** ale bus-ului (Moxa SW3 = OFF la capătul lui; terminație la invertorul de la capăt).
- ES200 interoghează fiecare echipament după **slave ID**; Multi Data Master agregă (vezi S3.5).

> Alternativă cu cost: dacă bus-ul lung e fragil sau invertoarele suportă TCP, poți merge TCP-per-invertor.

---

## 8. S3 — Acces local și activarea canalului Modbus/IEC104

**Vino echipat** cu aplicațiile de commissioning + credențialele site-ului. Dar **verifică pe loc** că ai acces **LOCAL pe echipament**, nu doar la portalul cloud (credențialele primite sunt frecvent conturi de cloud.

**Cum ajungi la configurarea unui echipament (în ordinea preferinței):**
1. **Web UI prin TCP/IP** — cel mai bun. Necesită IP default sau planul de rețea al investitorului.
2. **App pe USB (mini/micro-USB) sau WiFi** — FusionSolar etc.
3. **Ecran fizic + butoane** — limitat; de aici poți activa Modbus și seta unit ID / parametri de canal.

**Pași:** intri local → activezi canalul **Modbus/IEC104-slave** (e des
**dezactivat** by default de obicei) → citești parametrii reali (unit/slave ID,
baud/8N1, IP, port 502).

**NO-GO / escaladare:** acces doar cloud / blocat cu parolă de installer / ecran-only fără parola de service → escaladezi pentru credențialul corect sau installer. (Dar conform principiului — harvest + lasă cablat înainte de a pleca.)

---

## 9. S3.5 — Mai multe echipamente: agregare și distribuție

Northbound rămâne **mereu 2 RO + 2 RW**, indiferent de câte invertoare sunt.

- **Citire (RO):** ES200 **Multi Data Master** însumează → `P_SUMMED`, `Q_SUMMED`.
- **Scriere (RW):** un singur `SET_P` / `SET_Q` din northbound → **distribuit la fiecare echipament**:
  - **Procent (preferat):** dacă harta Modbus are setpoint **%-din-nominal** → scrii **același %** la toate → tot site-ul scalează împreună. Fără matematică de distribuție.
  - **Absolut:** dacă există doar valori absolute (kW/kvar) → distribui **proporțional cu puterea nominală** a fiecărui invertor (deci trebuie să cunoști rating-ul fiecăruia).

> Caută în register map întâi registrele de control în **procente** — simplifică enorm site-urile multi-invertor.

---

## 10. S4 — Validare: întâi direct prin ES200

1. **Fast path:** testezi **direct prin ES200** (preprovisionat).
   **Merge?** → site integrat, pleci la următorul site.
2. **Doar dacă sunt probleme** → scoți **laptopul** ca să izolezi *problemă de teren* vs *problemă de config*:
   - **Modbus Poll**, **QModMaster**, sau **Triangle Test Harness** (VM)
     pentru a testa conexiunea southbound.
   - Conexiune: **TCP** (laptopul **în aceeași rețea** cu echipamentul) **sau serial** prin **USB→DB9 (RS232)** (+driver, +COM-port corect), plus **Moxa** dacă e RS485.
   - Confirmi slave/unit IDs, găsești registrele reale P/Q + setpoint (%/absolut), validezi totul **înainte** de a încărca ES200.

---

## 11. S5 — Read + Write în siguranță

- **Read:** citește P și Q din **registrele reale** (confirmate, nu
  presupuse). Scalează registrii daca e cazul. La multi-device verifică
  suma în Multi Data Master.
- **Write (P):** testează pe **UN singur echipament** întâi, cu un **setpoint de test agreat cu operatorul**, și **verifică feedback-ul real** (producția chiar se modifică), nu doar „comandă acceptată”.
- **Write (Q):** P și Q se mapează **mereu**, dar **Q se testează live doar dacă** profilul DEER folosește efectiv reactiv **și** vendorul expune un registru reactiv sigur. Dacă Q nu e folosit de DEER sau e riscant → mapezi punctul, dar **nu** faci write live pe Q.
- **Multi-invertor pe același bus:** validează explicit că setpoint-ul ajunge la **toate** unitățile (Huawei: control care merge cu un invertor poate eșua cu mai multe pe același RS485).
- **Cloud:** confirmă că platforma existentă (FusionSolar/SolaX Cloud) **nu a fost afectată**.

---

## 12. S6 — Northbound IEC104 (în mare, remote)

Northbound se face **mai ales de la birou / remote**, nu în teren:

- ES200 e **preprovisionat** de acasă. DEER comunică în avans **ASDU / common address**, lista de **IOA** pentru cele 4 puncte, portul IEC104 (default **2404**) și **IP-ul master-ului FEP** (ca să-i dai allow).
- ES200 e stația **controlată** (server :2404); **FEP-ul DEER** (staging, apoi prod-core) e master-ul care o interoghează prin **APN-ul privat**.
- **Sarcina inginerului în teren = doar atât:** confirmă că **SIM-ul APN e sus** și că **echipa de networking ajunge prin VPN la IR1101 + aplicația ES200**. Restul (maparea IOA, cutover staging→prod, validarea VPP) se face **remote**.

---

## 13. S7 — Montaj și alimentare

**Termină stratul fizic la fiecare vizită** (e baza pentru „lasă cablat”):

- Montează pe **șină DIN** în dulapul de comunicații; dacă nu există → montezi o șină/bracket din kit.
- Alimentare din **24–48V DC** disponibil; dacă e doar **230V AC** → montezi o **sursă DIN 230VAC→24/48VDC** din kit.
- Asigură **port Ethernet liber** (switch/router/logger) dacă mergi pe TCP.
- Dacă montajul e imposibil (zero spațiu/alimentare) → tot **cablezi + documentezi** exact ce trebuie pregătit pentru follow-up.

---

## 14. Information harvest + as-built

La **fiecare** vizită, chiar și (mai ales) când nu termini:

- **Poze:** etichete echipamente, ecrane locale, dulap, porturi, cablaje finale.
- **Screenshot-uri** din web UI dacă ai TCP (config rețea, status Modbus, register list).
- **Diagnostice** Modbus/IEC104 (ce s-a citit/scris, slave IDs, erori).
- **As-built:** listă reală echipamente + putere; IP-uri finale; slave/unit IDs; baud/8N1; registre testate la read & write; tip setpoint (% / absolut); limitări cunoscute; ce mai e de făcut remote.
