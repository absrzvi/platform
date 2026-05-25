# ÖBB Role Research — Platform Relevance
**Date:** 2026-05-23
**Sources:** karriere.oebb.at, ts.oebb.at

---

## 1. Zugbegleiter:in — Conductor / Train Manager

**Source:** [karriere.oebb.at](https://karriere.oebb.at/de/karriereperspektiven/berufserfahrene/jobs-mit-kontakt-zum-fahrgast/zugbegleiterin)

### What they do
- First point of contact and host for passengers on regional and long-distance services
- Check and control tickets
- Provide schedule and pricing information
- Ensure passenger safety, comfort, and wellbeing
- Conduct vehicle inspections (brake tests) with the train operator
- Handle train dispatch procedures
- Execute operational announcements
- Coordinate operational workflows onboard
- Bear overall train responsibility (differs from SKT service roles)

### Work context
- 10.5-hour shifts, irregular schedule including nights/weekends/early mornings
- Moves continuously through coaches — rarely stationary
- Works alone or with one other crew member on most services
- Sole decision-maker onboard for passenger-facing operational issues

### Platform implications
- Moving through coaches = one-handed operation essential, glanceable UI
- Sole decision-maker = needs complete context, not partial data
- Ticket checking + passenger interaction = attention is split, alerts must interrupt clearly
- Brake tests + dispatch = safety-adjacent responsibilities, advisory LLM fits this mental model
- No reference to digital tools in current role description — this platform is a net-new workflow

---

## 2. Fahrdienstleiter:in — Traffic Controller / Train Dispatcher

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
- Coordination with train personnel = the dispatcher ↔ conductor communication loop our platform can reduce
- >50% adoption via UI (success criterion) is realistic for this role given existing digital work context

---

## 3. Disponent:in im Eisenbahnwesen — Rail Operations Dispatcher / Fleet Coordinator

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
- Fleet availability planning = needs predicted failure windows and maintenance scheduling data (FR28, FR29)
- Disruption response = the live crisis mode use case directly
- Cross-department coordination = intent packets with `request_landside_tasking: true` land here
- Cost management framing = "prevented delay-minutes" counterfactual evidence is directly valuable to this role
- This role is the operational Fleet Manager — distinct from the ECM maintenance manager

---

## 4. ECM Manager / Vehicle Technician — Maintenance Engineering

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
- **ECM 1 (manager)** = our primary persona. Arrives cold at depot, needs pre-arrival brief. Fears hidden faults. Bears regulatory accountability.
- **ECM 2** = maintenance documentation — our audit trail and work order export directly supports this function
- **ECM 3** = workshop oversight — our depot briefing shows which bay faults require which technician discipline
- **ECM 4 (technicians)** = the electrical/mechanical/IT split in our Depot Briefing role views maps exactly to real ÖBB technician specialisations
- Fault ACK trail in our interface is not just good UX — it supports the regulatory documentation obligation under EU ECM Directive
- "Authorises vehicles for operation" = the technician sign-off flow in our Depot Briefing is directly aligned with ECM 4's actual regulatory duty

---

## Key Insights for Platform Design

### 1. The technician role split is real and formally structured
Electrical, mechanical, IT/electronics are genuine ÖBB specialisations with different training paths. Our Depot Briefing role-differentiated views are not UX nicety — they map to how ÖBB actually organises its maintenance workforce.

### 2. ECM sign-off is a regulatory requirement, not just a workflow
EU Directive requires ECM authorisation before a vehicle returns to service. Our technician sign-off flow in the Depot Briefing gives the ECM Manager a digital audit trail that directly supports their regulatory obligation. This is a compliance feature, not just an operational one — position it that way in the commercial pitch.

### 3. The conductor has no existing digital tool for onboard decisions
The Zugbegleiter role description mentions no digital decision support tools. This platform is entirely additive — no workflow replacement, no change management friction. The "is it safe to continue?" advisory directly supports the conductor's existing brake test and dispatch responsibilities.

### 4. The dispatcher (Fahrdienstleiter) already works digitally
They operate signal/switch systems via computer monitors all day. Adding a fleet monitoring UI is a natural extension of their existing work context. >50% adoption target is realistic.

### 5. The Disponent is the operational Fleet Manager
The Disponent im Eisenbahnwesen is the role that maps to the Fleet Manager AI Interface user — fleet availability, disruption response, cross-department coordination. Distinct from the ECM Manager (maintenance focus). The Fleet Manager interface serves the Disponent's planning and disruption-response needs.

### 6. "Prevented delay-minutes" resonates with the Disponent's cost mandate
The Disponent is explicitly responsible for "cost management through strategic fleet deployment." The counterfactual evidence panel (prevented delay-minutes) in the Fleet Manager interface maps directly to this role's KPIs — not just a nice-to-have, a KPI they're already measured on.
