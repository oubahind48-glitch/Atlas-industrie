# Decision note - architecture of the Atlas Industrie campus

## 1. Context and requirements

Atlas Industrie operates two buildings: Building A (head office and IT) and Building B (production). The campus requires a resilient network architecture supporting 14 devices (2 cores, 2 firewalls, 10 access switches) with redundant fibre links between sites. Production must continue if building A is isolated.

## 2. Technical targets retained

- Dual-core architecture with cores in separate technical rooms (one per building)
- Two distinct fibre routes between buildings (north and south)
- Independent UPS per building
- VLAN segmentation: infrastructure, IT, voice, guests, OT, DMZ
- IPv4 and IPv6 dual-stack addressing
- 5-year horizon; power and cooling excluded from costing

## 3. The two architectures compared

**S1 - Three tiers**: 2 modular L3 cores + 4 L3 distribution switches + 10 access switches + 1 firewall pair. Full hierarchy, highest resilience, highest cost. 18 devices total.

**S2 - Collapsed core distributed between A and B**: 2 fixed 10/25G L3 cores (one per building) + 10 access switches + 1 firewall pair. Simpler hierarchy, true site-level resilience. 14 devices total.

**S3 - Single-site collapsed core (both cores in A)**: same device count as S2 (14 devices) but both cores located in building A. Cheapest, but fails the requirement that production continues if A is isolated. S3 is not an equivalent cheaper solution; it is a different solution that serves to price the requirement.

## 4. Five-year costing

Price assumptions (MAD, excluding tax, 1 EUR = 10.8 MAD):
- 48-port PoE+ access switch: 27,000 MAD each (10 units: 6 in A, 4 in B)
- High-end modular L3 core: 162,000 MAD
- Fixed 10/25G L3 core (collapsed core): 86,400 MAD
- L3 distribution switch: 54,000 MAD
- Firewall pair: 216,000 MAD (identical in the three scenarios)
- Optics and fibre: 129,600 MAD (three tiers) / 108,000 MAD (distributed core, second fibre on distinct route) / 86,400 MAD (single site)
- Vendor support: 10% of CAPEX per year
- Operations: 1.5 days per device per year at 5,400 MAD/day
- Horizon: 5 years. Power and cooling excluded.

Formulas:
- TCO(5 years) = CAPEX + 5 x (annual support + annual operations)
- annual support = 10% of CAPEX
- annual operations = number of devices x 1.5 d x 5,400 MAD

### S1 - Three tiers (18 devices)

| Item | Qty | Unit (MAD) | Total (MAD) |
|------|-----|------------|-------------|
| Modular L3 core | 2 | 162,000 | 324,000 |
| L3 distribution switch | 4 | 54,000 | 216,000 |
| 48-port PoE+ access switch | 10 | 27,000 | 270,000 |
| Firewall pair | 1 | 216,000 | 216,000 |
| Optics and fibre (three tiers) | 1 | 129,600 | 129,600 |
| **CAPEX** | | | **1,155,600** |

- Annual support: 115,560 MAD
- Annual operations: 18 x 1.5 x 5,400 = 145,800 MAD
- **TCO (5 years): 1,155,600 + 5 x (115,560 + 145,800) = 2,462,400 MAD**

### S2 - Collapsed core distributed A and B (14 devices) -- RETAINED

| Item | Qty | Unit (MAD) | Total (MAD) |
|------|-----|------------|-------------|
| Fixed 10/25G L3 core | 2 | 86,400 | 172,800 |
| 48-port PoE+ access switch | 10 | 27,000 | 270,000 |
| Firewall pair | 1 | 216,000 | 216,000 |
| Optics and fibre (distributed core) | 1 | 108,000 | 108,000 |
| **CAPEX** | | | **766,800** |

- Annual support: 76,680 MAD
- Annual operations: 14 x 1.5 x 5,400 = 113,400 MAD
- **TCO (5 years): 766,800 + 5 x (76,680 + 113,400) = 1,717,200 MAD**

### S3 - Single-site collapsed core, both cores in A (14 devices)

| Item | Qty | Unit (MAD) | Total (MAD) |
|------|-----|------------|-------------|
| Fixed 10/25G L3 core | 2 | 86,400 | 172,800 |
| 48-port PoE+ access switch | 10 | 27,000 | 270,000 |
| Firewall pair | 1 | 216,000 | 216,000 |
| Optics and fibre (single site) | 1 | 86,400 | 86,400 |
| **CAPEX** | | | **745,200** |

- Annual support: 74,520 MAD
- Annual operations: 14 x 1.5 x 5,400 = 113,400 MAD
- **TCO (5 years): 745,200 + 5 x (74,520 + 113,400) = 1,684,800 MAD**

### Decision gaps

- **S2 vs S1**: S2 saves 745,200 MAD over 5 years (2,462,400 - 1,717,200)
- **S2 vs S3**: S2 costs 32,400 MAD more over 5 years (1,717,200 - 1,684,800), i.e. 6,480 MAD/year

## 5. Decision and justification

**S2 (collapsed core distributed between A and B) is retained.**

**S3 is the cheapest. Why not retain it?** The answer is not "because it is worse", but that it does not satisfy a stated requirement: production must continue if building A is isolated. S3 is therefore not an equivalent cheaper solution; it is a different solution. It serves to price the requirement.

**Is that extra cost worth it?** The extra cost of S2 over S3 is 32,400 MAD over 5 years (6,480 MAD/year). With the assumption of 108,000 MAD per hour of outage, avoiding just 3.6 minutes of downtime per year (6,480 / (108,000/60)) justifies the investment. A figure in minutes speaks to management; a percentage of TCO tells it nothing.

## 6. Risks and points of failure

### Trap 1: Claiming 99.99%

Three devices in series at 99.9% give 0.999^3 = 0.997 (99.7%). Doubling the core and the firewall leaves the non-redundant access switch setting the ceiling. The result stays below the 99.99% target. To reach it, one would also need to analyse: power supplies, fibre routes, and failover mechanisms.

### Trap 2: Forgetting common causes

The parallel model assumes independent failures. Same room, same software version, same fibre route: these are not independent failures. The theoretical figure always overstates actual availability. In our design, both fibre routes share the pulling chamber under the courtyard -- a single excavation incident can cut both links simultaneously.

### Trap 3: Keeping quiet about exclusions

Power, cooling, civil works, network cards: if they are not priced, they appear in section 7 as excluded. An unmentioned item reads as a forgotten item.

## 7. Assumptions and limits

### Assumptions

- Fibre optic pricing based on three-tier model (129,600 MAD) for distinct routes
- Operations cost calculated at 1.5 days per device per year at 5,400 MAD/day
- Vendor support at 10% of CAPEX annually
- Outage cost assumption: 108,000 MAD per hour
- 5-year horizon without major technology refresh
- Prices are orders of magnitude; real project requires vendor quotations

### Limits (exclusions)

- Power and cooling costs excluded
- Civil works (trenching, cable chambers beyond the identified pulling chamber) excluded
- Network interface cards in servers and endpoints excluded
- Wireless network design not included
- Disaster recovery site outside the campus not considered
- Security policies (firewall rules, ACLs) to be detailed in subsequent sessions

---

*Author: Hind OUBA*
*Advanced Network Architectures - Computer Networks 2*
