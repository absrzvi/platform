# ÖBB Role Research — Landside Platform Relevance
**Date:** 2026-05-23
**Sources:** karriere.oebb.at, ts.oebb.at
**Scope:** Landside roles only. Conductor (Zugbegleiter) research lives in `oebb-brain/oebb-role-research-2026-05-23.md`.

---

## 1. Fahrdienstleiter:in — Traffic Controller / Train Dispatcher

**Source:** [karriere.oebb.at](https://karriere.oebb.at/de/karriereperspektiven/berufserfahrene/jobs-ohne-fahrgastkontakt/fahrdienstleiterin)

### What they do
- Monitor and coordinate railway traffic from control room — safety, punctuality
- Operate digital controls for signals and switches via computer systems
- Safety and emergency management
- Coordinate shunting operations and construction work
- Provide passenger updates about delays and service changes
- Communicate with train personnel and external contractors
- First contact for many operational matters during incidents

### Work context
- 5–12 hour shifts, rotating days/nights/weekends
- Desk-based, monitor-heavy environment
- High concentration demand — attention to detail, reaction time is critical
- Physical requirements: good health, hearing, no colour blindness

### Platform implications
- Already works with multiple monitors and digital systems — receptive to new UI
- Makes rapid decisions under time pressure — crisis mode entry state is critical
- "Passenger updates" = needs clear, plain-language incident summaries (not raw telemetry)
- Coordination with train personnel = the dispatcher ↔ conductor communication loop the platform can reduce
- >50% adoption via UI (success criterion) is realistic for this role given existing digital work context

---

## 2. Disponent:in im Eisenbahnwesen — Rail Operations Dispatcher / Fleet Coordinator

**Source:** [karriere.oebb.at](https://karriere.oebb.at/de/karriereperspektiven/berufserfahrene/wirtschaftliche-berufe/disponent-eisenbahnwesen)

### What they do
- Plan and assign rail vehicles and locomotive operators to services
- Coordinate maintenance and service schedules for fleet availability
- Monitor ongoing rail traffic and respond to disruptions
- Collaborate with workshops and vehicle maintenance teams
- Cost management through strategic fleet deployment
- Contribute to operational safety and punctuality

### Work context
- 38.5-hour week in 12-hour shifts (day/night rotation)
- Weekend and holiday availability required
- Solution-focused, high responsibility, stress-resilient profile
- Works across departments — workshop, maintenance, operations

### Platform implications
- Fleet availability planning = needs predicted failure windows and maintenance scheduling data
- Disruption response = the live crisis mode use case directly
- Cross-department coordination = IntentPackets with `request_landside_tasking: true` land here
- Cost management framing = "prevented delay-minutes" counterfactual evidence is directly valuable to this role
- This role is the operational Fleet Manager — distinct from the ECM maintenance manager

---

## 3. ECM Manager / Vehicle Technician — Maintenance Engineering

**Sources:** [ts.oebb.at/en/career/job-profiles](https://ts.oebb.at/en/career/job-profiles) | [ts.oebb.at/en/products-services/services/ecm-management](https://ts.oebb.at/en/products-services/services/ecm-management)

### ECM Management — four sub-functions

**ECM 1 — Overall maintenance oversight:**
- Overall responsibility for reliable maintenance and repair of vehicles
- Coordinates all maintenance functions, manages subcontracting
- Risk assessment in maintenance processes
- Scale: currently manages ECM responsibility for 20,000+ vehicles

**ECM 2 — Technical standards development:**
- Creates maintenance guidelines and technical documentation
- Manages design records, operator log books, maintenance logs
- Develops technically advanced maintenance and repair guidelines

**ECM 3 — Workshop oversight:**
- Technically approves repair facilities
- Monitors maintenance provider performance through control measures

**ECM 4 — Hands-on maintenance:**
- Performs actual maintenance work
- Documents all maintenance activities
- Authorises vehicles for operation after maintenance

### Vehicle Technician specialisations
- Electrical engineering (cable faults, power systems, camera power)
- Electronics / mechatronics (door motors, sensors)
- Mechanical engineering (brakes, HVAC, door mechanisms)
- IT / system technology (firmware, SNMP, network ports, maintenance digitisation)
- Refrigeration engineering (HVAC)

### Work context
- 4,000 technicians across ÖBB fleet
- EU Directive requires ECM to be defined and registered per vehicle
- Safety certification is a regulatory obligation — vehicle cannot operate without ECM sign-off
- Depot-based, works to arrival schedules

### Platform implications
- **ECM 1 (manager)** = primary persona for Depot Briefing Interface. Arrives cold at depot, needs pre-arrival brief. Fears hidden faults. Bears regulatory accountability.
- **ECM 2** = maintenance documentation — the audit trail and work order export directly supports this function
- **ECM 3** = workshop oversight — the depot briefing shows which bay faults require which technician discipline
- **ECM 4 (technicians)** = the electrical/mechanical/IT split in the Depot Briefing role views maps exactly to real ÖBB technician specialisations
- Fault ACK trail in the interface is not just good UX — it supports the regulatory documentation obligation under EU ECM Directive
- "Authorises vehicles for operation" = the technician sign-off flow in the Depot Briefing is directly aligned with ECM 4's actual regulatory duty

---

## 4. Teil-Projektleitung Telematik — Sub-Project Manager Telematics

**Source:** Comprehensive Research Pack — ÖBB Telematics Fleet Management & Automation (Gemini, May 2026)

### Organisational placement
ÖBB-Personenverkehr AG → Flottenmanagement → System- und Betriebstechnik → Team Telematik → Teil-Projektleitung Telematik (currently owning Cityjet DOSTO Neu / Cityjet Enzo programmes).

### What they do
- Coordinate at the intersection of onboard Operational Technology (OT) and landside Information Technology (IT)
- Orchestrate two capital-intensive fleet programmes:
  - **Cityjet Doppelstock Neu (Stadler KISS)** — €1.5B for 109 EMUs, type-approved 2026-04-21, rolling into Ostregion service
  - **Cityjet Enzo** — retrofit programme for Desiro ML / Talent EMUs, adding IoT edge gateways, 5G multi-carrier bonding, AI-enabled APC vision
- Write technical requirements sheets (Lastenhefte) for procurement
- Arbitrate Change Requests across vendor contracts
- Run V-model test structures during commissioning

### Educational and technical profile
- Advanced degree (HTL, FH, university) in telecommunications, software, electrical, or network engineering
- Deep expertise in Ethernet topologies, RSTP, LTE/5G link bonding, virtualization, edge constraints
- Public procurement and contract arbitration

### Daily routine (refined)
- 08:30 — Asset & comms check across both fleets for overnight dropouts, version mismatches, edge-fault alerts
- 10:30 — Technical alignment with system technicians on standardised configs and OTA policies
- 13:30 — Interdisciplinary project board with procurement, project leads, workshop managers
- 15:30 — PoC review using telemetry from active test vehicles / digital twin

### Primary pain points
1. **Vendor "black box" lock-in** — Stadler and Siemens protect proprietary bus data; deep diagnostic access triggers commercial Change Requests
2. **Sustained verification stress during Probebetrieb** — a single telematics breakdown (PIS mismatch, APC drop) freezes formal commissioning during the 2-week active passenger trial
3. **Configuration drift across the fleet** — vehicles entering different workshops (Simmering, Floridsdorf, Linz) receive inconsistent patch levels
4. **Connectivity droop in alpine tunnels** — distinguishing hardware link failure from geographical blackout is hard without geospatial overlay telemetry

### Platform implications
- This role is the primary persona for the landside platform — the daily user
- Workflow is fragmented across **Vendor portals**, **Central Diagnostics (Diagnoseplattform / Power BI)**, **ServiceNow (ITSM)**, **SAP PM/EAM (asset registry)**, **MS Project**, **Jira/Confluence**, **Excel** — platform must unify these or coexist cleanly with them
- Needs: fleet compatibility matrix, geospatial telemetry overlay, configuration drift detection, OTA rollout orchestration with phased validation, end-to-end network path visibility for vendor warranty triage

---

## 5. Application Landscape (current state — what the platform must integrate with or replace)

| System | Type | Owned by | Role in workflow |
|---|---|---|---|
| Vendor Diagnostic Portals | OT | Stadler / Siemens | Raw vehicle log files, bus controller parameters — proprietary |
| Central Diagnostics Platform (Diagnoseplattform) | OT | ÖBB | High-level uptime, passenger counting, fleet availability — Power BI fronted |
| ServiceNow | ITSM | ÖBB IT | Service desk, landside network failures, back-office routing |
| SAP PM/EAM | Asset | ÖBB Technische Services | Software builds linked to physical serial numbers, G-Check work orders |
| MS Project | Engineering | ÖBB project teams | Milestone schedules |
| Jira / Confluence | Engineering | ÖBB IT | Software sprint tracking |
| Excel | Engineering | Various | Hardware revision matrices, version reconciliation |
| HAFAS | Timetable | ÖBB | Real-time scheduled stops, delay data |
| BOOM | Work orders | ÖBB | Depot work order system (descoped for PoC) |

---

## Key Insights for Landside Platform Design

### 1. The technician role split is real and formally structured
Electrical, mechanical, IT/electronics are genuine ÖBB specialisations with different training paths. The Depot Briefing role-differentiated views are not UX nicety — they map to how ÖBB actually organises its maintenance workforce.

### 2. ECM sign-off is a regulatory requirement, not just a workflow
EU Directive requires ECM authorisation before a vehicle returns to service. The technician sign-off flow in the Depot Briefing gives the ECM Manager a digital audit trail that directly supports their regulatory obligation. This is a compliance feature — position it that way in the commercial pitch.

### 3. The dispatcher (Fahrdienstleiter) already works digitally
They operate signal/switch systems via computer monitors all day. Adding a fleet monitoring UI is a natural extension of their existing work context. >50% adoption target is realistic.

### 4. The Disponent is the operational Fleet Manager
The Disponent im Eisenbahnwesen is the role that maps to the Fleet Manager AI Interface user — fleet availability, disruption response, cross-department coordination. Distinct from the ECM Manager (maintenance focus).

### 5. "Prevented delay-minutes" resonates with the Disponent's cost mandate
The Disponent is explicitly responsible for "cost management through strategic fleet deployment." The counterfactual evidence panel maps directly to this role's KPIs — not a nice-to-have, a KPI they're already measured on.

### 6. Teil-Projektleitung Telematik is the integrator persona
The Gemini research identified a persona prior versions of this document missed: the Telematics Sub-Project Manager. They sit at the OT/IT boundary, own commissioning trial runs, coordinate vendor warranty triage, and run OTA rollouts. The 3-zone control board concept (Macro situational awareness → Micro train topology → Kondukt AI console with Action Intent cards) was designed around this persona. Pending wds-0 alignment, this is likely the primary daily user of the landside platform.
