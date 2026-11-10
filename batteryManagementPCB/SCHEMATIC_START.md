# CARL battery module: schematic starting plan

This is a learning plan for the existing KiCad project, not a build-ready electrical design. The sealed Ingeosolly Ingo-24V50 retains its internal BMS. The external module measures battery current and voltage, controls separate charge and drive paths, and reports over CAN.

## First sheet: external connections

Place four generic connector symbols on the root schematic. Use two pins for J1 through J3 and three pins for J4. The pin columns below describe the electrical signals that will be connected later. Do **not** edit the `Pin Functions` tab or rename `Pin_1` and `Pin_2` in the symbol properties. For this exercise, edit only the `Value` field on the `General` tab:

| Reference | Value | Pin 1 | Pin 2 | Pin 3 |
| --- | --- | --- | --- | --- |
| J1 | BATTERY | BATT_POS | BATT_NEG | — |
| J2 | CHARGER | CHARGER_POS | CHARGER_NEG | — |
| J3 | VEHICLE_POWER | VEHICLE_POS | VEHICLE_NEG | — |
| J4 | CARL_CAN | CAN_H | CAN_L | CAN_GND |

Connector pin numbers are provisional. Actual connector pinout and polarity must be checked when hardware is selected. Do not connect battery positive directly to vehicle positive or charger positive in the schematic.

## Planned power path

```text
J1 BATT_POS -- F1 -- R1 battery-current shunt -- BATT_BUS
BATT_BUS -- SW1 drive disconnect placeholder -- J3 VEHICLE_POS
BATT_BUS -- SW2 charge disconnect placeholder -- charger protection -- J2 CHARGER_POS
J1 BATT_NEG ------------------------------ J3 VEHICLE_NEG / J2 CHARGER_NEG
```

The battery-current sensor must be between the battery and **every** charge or discharge branch, including the board's own power supply. A second sensor in the charger branch measures charger current independently. SW1 and SW2 are schematic-only placeholders for controlled, bidirectionally blocking switches; their final implementation is undecided and they have no selected footprints. A precharge path may be needed around SW1 for vehicle input capacitors. The battery fuse F1 belongs physically close to the battery. The charge connector also needs its own fault protection.

## Planned signal blocks

- Battery voltage sensing: BATTERY_BUS and BATT_NEG to a protected measurement circuit.
- Battery case thermistor: sensor connector to temperature measurement circuit.
- Charger-present detection: charger connector to a hardware drive inhibit and to Teensy status input.
- Independent fault logic: overcurrent, bad charger input, temperature fault, or missing control power opens the affected switch.
- Teensy 4.1: reads measurements and switch feedback; sends CAN controller signals to a CAN transceiver.
- CAN transceiver: connects to J4 CAN_H, CAN_L, and CAN_GND. CAN termination is populated only if this node is at an end of CARL's CAN bus.
- Power supply: protected BATTERY_BUS input to regulated Teensy and sensor power. The Teensy must not receive pack voltage directly.

## Values intentionally unresolved

The battery label confirms 25.6 V nominal and 50 Ah, but does not specify charge current, continuous/peak discharge current, short-circuit current, or internal BMS thresholds. The third-party review's 50 A and 28.8 V figures are provisional. CARL motor starting current, controller input capacitance, charger rating, wire size, and CAN topology are also needed before selecting or rating components.

## Confirmed CARL supply behavior

The existing CARL 5 V fusebox is switched off when the vehicle is off and the battery is charging. It draws from the same 24 V battery but bypasses the R1 current-sense path shown on this schematic. Therefore it cannot be the sole supply for live charging telemetry, and R1 cannot count that fusebox load in the state-of-charge estimate as currently wired. Keep the dedicated `AUX_5V_SUPPLY` block fed from `BATT_BUS`. Before the design is finalized, either route the fusebox's battery feed through the common battery current sensor or measure its branch separately. Do not claim accurate coulomb-counted state of charge until every battery branch is accounted for.

The `AUX_5V_SUPPLY` sheet currently has input and output capacitors drawn around a future wide-input 5 V converter. The MORNSUN K78U05-1000R3 is a candidate (9-75 V input, 5 V/1 A output), not yet a finalized part. Its datasheet calls out extra heat-dissipation shell terminals and states that the input is not reverse-polarity protected. The project-specific symbol and eventual footprint must capture the actual package and all terminals; do not substitute a plain three-pin linear-regulator footprint. See the manufacturer's K78Uxx-1000R3(L) datasheet.

## First KiCad exercise

1. Open `batteryManagementPCB.kicad_sch` in Schematic Editor.
2. Press **A** and search for `Conn_01x02` in `Connector_Generic`; place three copies, spaced apart. The `_Pin` variant also works for this exercise.
3. Press **A** and search for `Conn_01x03`; place one copy for CAN.
4. Hover over each connector and press **E**. Stay on the **General** tab and edit the field named **Value** to `BATTERY`, `CHARGER`, `VEHICLE_POWER`, or `CARL_CAN`. Keep the automatic **Reference** field if KiCad assigned different J numbers. Leave the **Pin Functions** tab alone.
5. Later, add short wires from the connector pins and attach net labels (`L`) to the wire ends. Net labels are separate schematic objects; they are not entered into the connector's Value or Pin Functions fields. Verify the pin numbers on the drawing before assigning signals.
6. Save. This first step defines the module boundary. Do not wire the three positive connector pins together.
