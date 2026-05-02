# 2. System Design & Architecture

## Diagrams (8 files)

All diagrams are in Draw.io format and can be opened in the [Draw.io editor](https://draw.io) or exported to other formats.

**Architecture Diagrams:**
1. [System-Architecture.drawio](System-Architecture.drawio) - Hub-centric system topology with all components and connectivity layers
2. [Component-Interactions.drawio](Component-Interactions.drawio) - Data flows and event processing pipeline
3. [Deployment-Diagram.drawio](Deployment-Diagram.drawio) - Physical layout across 6 home zones with wireless coverage

**SysML Diagrams:**
4. [System-Blocks.drawio](System-Blocks.drawio) - 8 major system blocks and internal interfaces
5. [Use-Cases.drawio](Use-Cases.drawio) - 15 use cases for 4 actor types (Homeowner, Resident, Technician, Cloud Operator)
6. [State-Machines.drawio](State-Machines.drawio) - 4 state machines (System, Connectivity, Power, Alert Processing)
7. [Sequence-Diagrams.drawio](Sequence-Diagrams.drawio) - 6 critical scenarios with timing (Motion Detection, Arm Command, WiFi Failover, Video on Cellular, Door Alarm, Cloud Down)
8. [Requirements-Traceability.drawio](Requirements-Traceability.drawio) - Bidirectional mapping of requirements to components and test cases

## Summary

- **Hub-centric architecture**: Local processing with Raspberry Pi 4B hub, 2TB storage, 4+ hour battery backup
- **Components**: Sensors (Active IR, Magnetic contacts, Resistive cord), Cameras (4K PoE primary + doorbell secondary), Controllers (WiFi + Z-Wave)
- **Connectivity**: WiFi primary, LoRaWAN mesh secondary, Cellular tertiary, Satellite optional
- **8 SysML blocks**: Control Unit, Sensors, Cameras, Alarm, Power, Connectivity, UI, Cloud Services
- **100% requirements traceability** to test cases

