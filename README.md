# Meshtastic PoE 1W Node for Residential Installation

A fixed outdoor Meshtastic node with a 1W amplifier and integrated LNA, powered
and backhauled over a **single CAT5E run**. All electronics — radio, Ethernet,
PoE extraction, and the RF amplifier — live in one outdoor enclosure. Only the
CAT5E cable crosses the building wall; the PoE injector and surge protector sit
indoors in the panelboard.

- **Radio:** RAK4631 (nRF52840, US915) on a RAK19001 base board
- **RF:** RigExpert 915MPA (30 dBm PA + LNA) → 8 dBi fiberglass antenna
- **Backhaul + power:** RAK13800 Ethernet + RAK9168 PoE, one CAT5E run
- **Injection:** indoor UACC-POE+-2.5G, protected by ETH-SP-G2 at building entry

---

## System Diagram

```
   OUTDOORS                                            INDOORS (panelboard)
 ═══════════════════════════════════════        ═══════════════════════════════

        ┌─────────────┐
        │  8 dBi      │
        │  Fiberglass │
        │  Antenna    │
        │  915 MHz    │
        └──────┬──────┘
               │ N-male
               │ LMR-400 (3 ft)          ⏚ ground rod
               │                         │
   ┌───────────┴──────────────────┐      │ #10 AWG
   │ OUTDOOR ENCLOSURE  1555WA2GY │──────┘  via ILSCO GBL-4DBT clamp
   │ (IP67, holds all electronics)│
   │                              │
   │   ⊙ N-female bulkhead        │
   │   │                          │
   │   │ RG-316 MMCX↔N pigtail    │
   │   │                          │
   │  ┌┴──────────────────┐       │
   │  │ RigExpert 915MPA  │       │      RF stays entirely outdoors.
   │  │ 30 dBm PA + LNA   │       │      Nothing RF crosses the wall.
   │  │ 5 VDC · MMCX ports│       │
   │  └┬──────────────────┘       │
   │   │ IPEX↔MMCX pigtail        │
   │  ┌┴──────────────────────┐   │
   │  │ RAK19001 base board   │   │
   │  │  ┌─────────┐ ┌──────┐ │   │
   │  │  │ RAK4631 │ │RAK   │ │   │
   │  │  │ US915   │ │13800 │ │   │
   │  │  │ MCU/LoRa│ │ Eth  │ │   │
   │  │  └─────────┘ └──┬───┘ │   │
   │  │  ┌───────────┐  │     │   │
   │  │  │ RAK9168   │  │     │   │
   │  │  │ PoE (5V)  │  │     │   │
   │  │  └─────┬─────┘  │     │   │
   │  └────────┼────────┼─────┘   │
   │      5V   │  RJ45  │         │
   │   power ◄─┘        │ data    │
   │           (both from CAT5E)  │
   └───────────────────┬──────────┘
                       │ CAT5E (data + PoE)
                 ┌─────┴─────┐
                 │ PG11 gland│  cable entry, enclosure wall
                 └─────┬─────┘
                       │
 ══════════════════════╪════════ building wall ═══╪══════════════════════════
                       │  CAT5E only through wall  │
                       │                           │
                       │              ┌────────────┴───────────┐
                       └──────────────►  ETH-SP-G2             │
                          building     │  Ethernet surge prot. │──⏚ building
                          entry point  │  (PoE pass-through)   │   ground
                                       └────────────┬──────────┘
                                                    │ CAT5E (data + PoE)
                                       ┌────────────┴───────────┐
                                       │  UACC-POE+-2.5G        │
                                       │  Indoor PoE Injector   │
                                       │  30 W                  │
                                       └────────────┬───────────┘
                                                    │ LAN (data)
                                       ┌────────────┴───────────┐
                                       │  UDM-SE                │
                                       │  Router / Gateway      │
                                       │  VLAN 20 · 10.20.0.0/24│
                                       └────────────────────────┘
```

> **PoE direction:** the indoor UACC-POE+-2.5G injects power onto the CAT5E.
> Power flows *outward* to the enclosure, where the RAK9168 extracts 5 V.
> Data flows both ways. The ETH-SP-G2 is PoE-pass-through rated and sits at the
> building entry so surges are clamped before reaching indoor gear.

---

## RF Chain

All RF is contained inside the outdoor enclosure and its external antenna. No RF
connector crosses the building wall.

```
 RAK4631          915MPA (MMCX)                    N-female         LMR-400        8 dBi
  IPEX ──IPEX↔MMCX──► radio ═══ PA + LNA ═══ ant ──MMCX↔N──► bulkhead ══════►  antenna
         pigtail      port      +15 dB TX      port  (RG-316)  (encl. wall)    (3 ft)   915 MHz
```

**Link budget (TX):**

```
   RAK4631 output     +15 dBm   (firmware-limited)
   915MPA gain        +15 dB
   ───────────────────────────
   Conducted power    +30 dBm   (1 W)
   Antenna gain       + 8 dBi
   ───────────────────────────
   EIRP               +38 dBm
```

---

## Power & Data — Single CAT5E Run

```
   INDOOR                                                    OUTDOOR
   UDM-SE ──data──► UACC-POE+-2.5G ──► ETH-SP-G2 ──►│ wall │──► RAK9168 ──► 5V ──┬─► RAK19001
                    (inject 30W PoE)   (surge, entry) CAT5E     (PoE extract)    └─► 915MPA
                                                      only
```

> 🔧 **In progress — amp power distribution.** How the 915MPA is fed 5 V from the
> RAK9168 rail (splitter, dedicated buck, bulk capacitance) is not yet finalized.
> See [Open Issues](#open-issues).

**Power budget @ 5V:**

```
   915MPA TX peak     ~1.96 A   ████████████████████░░░░░░░░
   RAK4631 + board    ~0.15 A   █░
   RAK13800           ~0.10 A   █
   ───────────────────────────
   Peak total         ~2.21 A   ██████████████████████░░░░░░
   RAK9168 ceiling     6.00 A   ████████████████████████████████████████████████████████████
   Headroom           ~3.79 A
```

---

## Network Topology (UDM-SE)

The node lives on an isolated IoT VLAN with no path back into the trusted LAN.

```
   ┌──────────────────────────────────────────────────────────────┐
   │  UDM-SE                                                        │
   │                                                                │
   │   WAN ◄────────────────────────────┐  allow RF-IoT → WAN      │
   │                                     │                          │
   │   LAN 192.168.1.0/24  ──╳── blocked ┤  block RF-IoT → LAN      │
   │        │                            │  (except mgmt below)     │
   │        │  mgmt: 192.168.1.x ──► 10.20.0.10:80  (left open)     │
   │        │                            │                          │
   │   VLAN 20  RF-IoT  10.20.0.0/24  gw 10.20.0.1                  │
   │        │                                                        │
   │   access port (feeds the indoor PoE injector), VLAN 20         │
   └────────┼───────────────────────────────────────────────────────┘
            │
            ▼
      RAK13800  ──►  DHCP reservation  ──►  10.20.0.10
```

---

## Bill of Materials

*Source of truth: [BOM.xlsx](BOM.xlsx).*

| Component | Part Number / Model | Description | Qty |
|---|---|---|---|
| WisBlock Base Board | RAK19001 | Motherboard with dual IO slots | 1 |
| WisBlock Core (MCU/LoRa) | RAK4631 | nRF52840 MCU running Meshtastic firmware | 1 |
| LoRa Booster (1-Watt) | RigExpert 915MPA | 30 dBm PA with integrated LNA, 5 VDC | 1 |
| Ethernet Module | RAK13800 | 10/100 Mbps RJ45 interface | 1 |
| PoE Power Module | RAK9168 | IEEE 802.3af PoE extraction | 1 |
| Outdoor Enclosure | 1555WA2GY | IP67 waterproof housing + mounting plate | 1 |
| Internal RF Pigtail | RG-316 MMCX to N-Female | Bridges booster (MMCX) to enclosure-wall bulkhead | 1 |
| Exterior Low-Loss Coax | LMR-400 (3 ft) | Double-shielded low-loss cable | 1 |
| Indoor PoE Injector | UACC-POE+-2.5G | 30 W adapter, indoors in panelboard | 1 |
| High-Gain Antenna | 8 dBi Fiberglass (915 MHz) | Omni-directional | 1 |
| Ethernet Surge Protector | Ubiquiti ETH-SP-G2 | Gigabit, PoE pass-through, at building entry | 1 |
| Weatherproofing Tape | 3M 2228 Rubber Mastic | Over N-type junction, then LMR-400 jacket | 1 |
| Cable Entry Gland | PG11 Nylon Gland | Clamps 5–10 mm; CAT5E OD ~5.5–6.2 mm | 1 |
| Ground Wire | #10 AWG | By the foot | 1 |
| GEC Clamp | ILSCO GBL-4DBT | Irreversible clamp to ground rod / GEC | 1 |

---

## Grounding

- **Outdoor enclosure / antenna side:** #10 AWG bonded via ILSCO GBL-4DBT
  irreversible clamp to the ground rod / GEC.
- **Indoor entry:** ETH-SP-G2 at the point where CAT5E enters the building,
  ground lug to building ground.
- **Weatherproofing:** 3M 2228 mastic tape over the N-type connector junction,
  then over the LMR-400 jacket.

---

## Meshtastic Firmware Settings

| Setting | Value |
|---|---|
| Region | `US_915` |
| TX power | `15 dBm` |
| Role | `Router` |
| Primary channel PSK | set on deployment |
| Admin channel PSK | set on deployment, separate from primary |
| MQTT | configure if using a local broker |

---

## Open Issues

- [ ] 🔧 **5V amp power distribution — IN PROGRESS.** Define how the 915MPA is fed
      5 V from the RAK9168 rail: splitter vs. dedicated buck, wire gauge, and bulk
      capacitance to hold up the ~1.96 A TX peak. *Actively being worked.*
- [ ] **915MPA power connector type** — confirm 5 VDC input connector.

---

*[BOM.xlsx](BOM.xlsx) is the authoritative source for parts.*
