# Hardware-Box telemetry appliance — reference concept

**Status:** conceptual low-voltage reference architecture. Not a fabrication-ready PCB, wiring diagram or certified electrical design.

## Mission

A read-only edge appliance for the laboratory and data-centre monitoring workflows. It collects environmental and equipment telemetry, validates readings locally, buffers them during network outages and forwards signed, timestamped data to Flare-Hunt / Asimov-Meter / Safe-Break.

It must not act as a robot controller, breaker controller or autonomous power switch.

## System block sketch

    ┌────────────────────── CERTIFIED DC ADAPTER ──────────────────────┐
    │                         isolated low-voltage DC                  │
    └────────────────────────────┬─────────────────────────────────────┘
                                 │
                       [Input protection]
                       fuse + reverse-polarity
                       + transient protection
                                 │
                         [DC/DC regulators]
                           │             │
                        5 V rail      3.3 V rail
                           │             │
                   ┌───────▼─────────────▼────────┐
                   │     COMPUTE / CONTROL        │
                   │                              │
                   │  MCU / industrial SBC        │
                   │  Secure element              │
                   │  Watchdog + RTC              │
                   │  Encrypted local buffer      │
                   │  Signed firmware updates    │
                   └───────────────┬──────────────┘
                                   │
                    ┌──────────────┼─────────────────┐
                    │              │                 │
              I²C / SPI       isolated input     Ethernet
                    │              │                 │
          ┌─────────▼──────┐ ┌─────▼────────┐ ┌──────▼────────┐
          │ temp / humidity│ │ dry contact  │ │ TLS telemetry │
          │ vibration      │ │ leak sensor  │ │ MQTT / HTTPS  │
          │ fan tach       │ │ isolated     │ │ outbound only │
          └────────────────┘ │ pulse counter│ └──────┬────────┘
                             └──────────────┘        │
                                              Asimov-Meter /
                                              Flare-Hunt /
                                              Safe-Break

## Connector and signal plan

| Connector | Intended signal | Boundary |
|---|---|---|
| J1 | DC input from certified adapter | Low-voltage DC only; adapter handles mains isolation |
| J2 | I²C sensor bus | Temperature/humidity sensors with documented voltage levels |
| J3 | Tachometer / pulse input | Protected digital input; isolation where field wiring requires it |
| J4 | Dry-contact input | Leak/door state; debounced and fault-monitored |
| J5 | Isolated field interface | Use a certified isolated transceiver for external/long cable runs |
| J6 | Ethernet | Outbound TLS telemetry; no default inbound administration |
| J7 | Service USB | Physical-access-only maintenance mode |

No pin numbers, trace widths, fuse ratings or component part numbers are frozen. They must be derived from the selected parts and measured load.

## Conceptual board floorplan sketch

    +----------------------------------------------------------------+
    | J1 DC IN | Protection | DC/DC | 5V/3V3 rails                  |
    |----------------------------------------------------------------|
    | J2 I2C header      MCU + RTC + watchdog      Secure element    |
    |                                                                |
    | J3 tach input     J4 dry contact    J5 isolated field input   |
    |                                                                |
    |              encrypted storage / local event buffer            |
    |----------------------------------------------------------------|
    | J6 Ethernet + magnetics              J7 service connector       |
    +----------------------------------------------------------------+

Keep the power entry/protection region separated from low-level analogue sensor routing. Final PCB stack-up, creepage/clearance, return paths, shielding, connector ratings and thermal design require an electrical engineer’s review.

## Illustrative BOM (categories, not selected part numbers)

| Ref | Category | Purpose | Selection evidence required |
|---|---|---|---|
| PS1 | Certified external DC adapter | Safe power source | Safety certification, output rating, region approval |
| F1 | Input fuse / protected input module | Over-current protection | Worst-case current and inrush calculation |
| U1 | MCU or industrial SBC | Control and buffering | Lifecycle, watchdog, security update support |
| U2 | Secure element | Device identity / key storage | Documented key lifecycle and provisioning |
| U3 | DC/DC regulator | Stable rails | Thermal and transient validation |
| U4 | RTC | Timestamp continuity | Drift and backup-power characteristics |
| J2 sensors | Temperature / humidity / vibration | Room and equipment telemetry | Calibrated measurement range |
| J3/J4 | Protected input circuits | Fan tach / dry contacts | Voltage, filtering and isolation review |
| J5 | Isolated transceiver | External sensor links | Isolation rating and EMC test |
| J6 | Ethernet interface | Network telemetry | ESD and isolation requirements |

## Firmware / data flow

1. Read sensor with a timestamp and device identity.
2. Validate value range, checksum and sensor-health flags.
3. Mark stale/missing/out-of-range readings as faults; never substitute a synthetic value.
4. Buffer records locally using an append-only sequence number.
5. Sign the batch or device envelope and transmit over TLS.
6. Retain an audit trail of firmware version and calibration metadata.

## Safety and security gates

- No direct mains wiring on the prototype board. Use an externally certified low-voltage DC adapter.
- No remote actuation or power switching.
- Signed firmware updates, rollback image, watchdog and secure boot where supported.
- Outbound-only network policy by default.
- Physical service access and tamper-evident enclosure.
- Environmental, EMC, ingress-protection and electrical safety tests before deployment.

## Next engineering deliverables

1. Freeze one use-case and sensor BOM.
2. Calculate current budget, rail transients and thermal dissipation.
3. Produce a real KiCad schematic and PCB after parts are selected.
4. Run ERC/DRC, bench tests, thermal tests and enclosure review.
5. Obtain independent electrical and regulatory review.

Hardware-Box already contains KiCad and CAD inspection utilities. This document is a proposed telemetry-box configuration; it does not replace real schematic capture or design-rule validation.
