# CoorenIO — Embedded Coordination (Wokwi Simulation)

**Domain**: Multi-zone smart thermostat / HVAC damper control

**Hardware simulated in Wokwi**:
- Arduino Mega 2560
- 5× DHT22 temperature sensors
- 4× motorized dampers (Master, Living Room, Bedroom, Kitchen) — LEDs represent damper state (open/closed)

**Runtime**: CoorenIO firmware compiled and running directly on the board.

## What Cooren Does

CoorenIO treats each zone as a **participant** that sends **signals** (temperature demand).  
It runs the same Listen-Act-Close coordination loop used in Dinner Decider and Project Jeda, then issues actuator decisions.

**Sample Session Log** (from running simulation):
[COOREN] Session created: THERM-001
[COOREN] Participants registered: 9   Setpoint: 68.00F
[ZONE] Demand triggered: Master @ 75.20F
... (other zones)
[COOREN] Decision: Zones demanding: 4
damper_Master -> OPEN
damper_LivingRoom -> OPEN
damper_Bedroom -> OPEN
damper_Kitchen -> OPEN


## Why This Example Matters

The identical six-operation coordination primitive now runs on constrained embedded hardware and directly controls physical actuators (simulated here in Wokwi).

**Note**: Full CoorenIO firmware source is proprietary and lives in a private repository (McLeod-Interactive-Group-LLC/CoorenIO). This example only demonstrates domain portability.

## Visuals


---

**Links**:
- [Cooren API Home](../README.md)
