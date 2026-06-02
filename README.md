# EV Charging Sequence Diagrams and State Machines

A collection of open-source UML sequence diagrams and finite-state machines for common EV charging protocols:

- **[SAE J1772 PWM Charging (Control Pilot)](./diagrams/sequence/pwm-charging/)**
- **[SAE J1772 PWM Controlled Charging with OCPP 1.6J](./diagrams/sequence/pwm-ocpp16/)**
- **[ISO 15118-2 Controlled Charging with OCPP 1.6J](./diagrams/sequence/iso15118_2_ac-ocpp16/)**
- **[ISO 15118-2 HLC Optimized Charge Scheduling with OCPP 2.0.1](./diagrams/sequence/iso15118_2_ac-ocpp2/)**
- **[SAE J1772 EVSE Control Pilot FSM](./diagrams/state-machine/evse-control-pilot/)**

## ISO 15118-20 + OCPP 2.1 Diagrams

- **[ISO 15118-20 DC Scheduled Charging + OCPP 2.1](./diagrams/sequence/iso15118_20_dc-ocpp21_scheduled/)**
  Full DC Scheduled charging session with PnC authentication, DC safety phases (CableCheck, PreCharge, WeldingDetection), schedule exchange, charge loop, and schedule renegotiation.

- **[ISO 15118-20 AC Scheduled Charging + OCPP 2.1](./diagrams/sequence/iso15118_20_ac-ocpp21_scheduled/)**
  AC Scheduled charging session using power-based parameters (Watts) with PnC authentication, no DC safety phases, and AC charge loop at 10–60 second intervals.

- **[ISO 15118-20 DC BPT + OCPP 2.1](./diagrams/sequence/iso15118_20_dc_bpt-ocpp21_dynamic/)**
  Bidirectional DC power transfer (V2G) with sequential charge and discharge phases, signed energy metering for V2G billing, NotifySettlement with net export credit.

- **[ISO 15118-20 AC BPT + OCPP 2.1](./diagrams/sequence/iso15118_20_ac_bpt-ocpp21_dynamic/)**
  Bidirectional AC power transfer (V2H) for residential applications using the EV's on-board inverter with Dynamic control mode.

## 📊 Diagram Comparison Overview

- **SAE J1772 PWM Charging (Control Pilot)**  
  Focuses solely on the low-level, analog control pilot and proximity handshake between EVSE and EV to establish a charge session, modulate current and end a session.

- **SAE J1772 PWM Controlled Charging with OCPP 1.6J**  
  Builds on the pure PWM sequence diagram by weaving in OCPP 1.6J messages between EVSE and CSMS for session management and grid-side coordination.

- **ISO 15118-2 Controlled Charging with OCPP 1.6J**  
  Uses ISO 15118-2:2013 over HomePlug GreenPhy (HPGP) Powerline Communication (PLC) for EV ↔ EVSE and OCPP 1.6J for EVSE ↔ CSMS. The CSMS can _push_ a charging profile to the EV, but there’s _no bidirectional negotiation_ to balance driver energy needs and departure times with grid constraints-vehicles simply follow the profile provided by the Secondary Actor via the CSMS.

- **ISO 15118-2 HLC Optimized Charge Scheduling with OCPP 2.0.1**  
  Retains the ISO 15118-2 HLC flows but upgrades to OCPP 2.0.1’s richer set of messages and _adds true negotiation_. EV, CSMS, and optionally a Secondary Actor exchange requirements and constraints so that the final schedule optimally meets both driver departure/energy needs and grid/operator limits.

- **SAE J1772 EVSE Control Pilot FSM**  
  A finite-state machine view of the SAE J1772 control pilot, ideal for understanding basic pilot-voltage state transitions.

## ℹ️ Acknowledgments & Attribution

The National Charging Experience Consortium, or [ChargeX Consortium](https://inl.gov/chargex/), is a collaborative effort between Argonne National Laboratory, Idaho National Laboratory, National Renewable Energy Laboratory, electric vehicle (EV) charging industry experts, consumer advocates, and other stakeholders.

These diagrams (and any derivatives thereof) were developed by the **ChargeX Consortium**, specifically **Argonne National Laboratory**.

If you publicly use, display, or redistribute them-or any modified versions-please include the following attribution in your documentation, presentations, or source:

> “Diagram(s) based on work by Argonne National Laboratory (www.anl.gov).”  
## 📂 Repository Structure

```text
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── diagrams/
   ├── sequence/
   │   ├── pwm-charging/
   │   │   ├── pwm-charging.puml    # PlantUML source
   │   │   ├── pwm-charging.svg     # Rendered diagram (editable in Visio)
   │   │   ├── pwm-charging.png     # Rendered diagram
   │   │   └── README.md            # Description of the diagram
   │   ├── iso15118_2_ac-ocpp2/
   │   │   ├── iso15118_2_ac-ocpp2.puml
   │   │   ├── iso15118_2_ac-ocpp2.svg
   │   │   ├── iso15118_2_ac-ocpp2.png
   │   │   └── README.md
   │   ├── iso15118_2_ac-ocpp16/
   │   │   ├── iso15118_2_ac-ocpp16.puml
   │   │   ├── iso15118_2_ac-ocpp16.svg
   │   │   ├── iso15118_2_ac-ocpp16.png
   │   │   └── README.md
   │   ├── pwm-ocpp16/
   │   │   ├── pwm-ocpp16.puml
   │   │   ├── pwm-ocpp16.svg
   │   │   ├── pwm-ocpp16.png
   │   │   └── README.md
   │   ├── iso15118_20_dc-ocpp21_scheduled/
   │   │   ├── iso15118_20_dc-ocpp21_scheduled.puml
   │   │   ├── iso15118_20_dc-ocpp21_scheduled.svg
   │   │   ├── iso15118_20_dc-ocpp21_scheduled.png
   │   │   └── README.md
   │   ├── iso15118_20_ac-ocpp21_scheduled/
   │   │   ├── iso15118_20_ac-ocpp21_scheduled.puml
   │   │   ├── iso15118_20_ac-ocpp21_scheduled.svg
   │   │   ├── iso15118_20_ac-ocpp21_scheduled.png
   │   │   └── README.md
   │   ├── iso15118_20_dc_bpt-ocpp21_dynamic/
   │   │   ├── iso15118_20_dc_bpt-ocpp21_dynamic.puml
   │   │   ├── iso15118_20_dc_bpt-ocpp21_dynamic.svg
   │   │   ├── iso15118_20_dc_bpt-ocpp21_dynamic.png
   │   │   └── README.md
   │   └── iso15118_20_ac_bpt-ocpp21_dynamic/
   │       ├── iso15118_20_ac_bpt-ocpp21_dynamic.puml
   │       ├── iso15118_20_ac_bpt-ocpp21_dynamic.svg
   │       ├── iso15118_20_ac_bpt-ocpp21_dynamic.png
   │       └── README.md
   └── state-machine/
       ├── evse-control-pilot/
           ├── evse-control-pilot.puml
           ├── evse-control-pilot.svg
           ├── evse-control-pilot.png
           └── README.md      
```````

## 🔧 Development Environment

1. **Install the PlantUML extension**  
   - In VS Code, open the Extensions view (`Ctrl+Shift+X` / `Cmd+Shift+X`) and install **PlantUML** by **jebbs**.
2. **Install prerequisites**  
   - **Java JDK** (version 8 or later) for PlantUML’s runtime.  
   - **Graphviz** (https://graphviz.org/) to render diagrams.
3. **Configure PlantUML in VS Code**  
   - Open Settings (`Ctrl+,` / `Cmd+,`) and search for `plantuml`.  
   - If using a local PlantUML JAR, set **`plantuml.server`** to the file path (e.g., `"/path/to/plantuml.jar"`) and ensure `java` is in your system `PATH`.  
   - Set **`plantuml.exportFormat`** to `svg` (or `png`) as preferred.
   - Set **`plantuml.commandArgs`** to `-DPLANTUML_LIMIT_SIZE=16384`. PlantUML's default 4096 px limit silently clips the taller ISO 15118-20 sequence diagrams; the override lets them render at full height. If rendering from the CLI instead, pass the same JVM flag: `java -DPLANTUML_LIMIT_SIZE=16384 -jar ~/plantuml.jar -tpng file.puml`.
4. **Preview and export diagrams**  
   - Open any `.puml` file and press `Alt+D` to launch the PlantUML preview.  
   - Use the export buttons in the preview toolbar to generate `.svg` or `.png` files directly.

## 🤝 Contributing

- Fork, add your diagram or update existing, submit a PR.  
- Please follow the naming conventions in the folder structure.  
- See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## 📄 License

This project is licensed under Apache 2.0. See [LICENSE](LICENSE) for details.
