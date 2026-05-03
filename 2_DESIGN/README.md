# 2. System Design & Architecture

## Diagrams (8 files)

All diagrams are in Draw.io format and can be opened in the [Draw.io editor](https://draw.io) or exported to other formats.

**Architecture Diagrams:**

1. [System-Architecture.drawio](System-Architecture.drawio) - Hub-centric system topology with all components and connectivity layers
2. [Component-Interactions.drawio](Component-Interactions.drawio) - Data flows and event processing pipeline
3. [Deployment-Diagram.drawio](Deployment-Diagram.drawio) - **Physical layout across 6 home zones with wireless coverage annotations**
   - **NEW**: Distance labels showing hub-to-zone distances (all zones < 10m in typical deployment)
   - **NEW**: REQ-012 compliance badge verifying ≥100m sensor range requirement
   - **NEW**: Coverage analysis legend explaining wireless requirement verification
   - Worst-case sensor distance: 8.21m from hub (Zone 3: Back Door/Patio)
   - All zones easily within 100m line-of-sight requirement (91.79m safety margin)

**SysML Diagrams:** 4. [System-Blocks.drawio](System-Blocks.drawio) - **SysML Internal Block Diagram (IBD): 8 major system blocks with ports, interfaces, and internal structure**

- Formal SysML notation with required/provided ports
- Shows Control Unit, Sensors, Cameras, Alarm, Power, Connectivity, UI, Cloud Services blocks
- Internal connections and interface specifications
- Reference for System Context Diagram (mermaid in System-Requirements-Specification.md)

  4.1. [Control-Unit-IBD.drawio](Control-Unit-IBD.drawio) - **Hierarchical Decomposition: Control Unit internal structure (one level deeper than System-Blocks)**

- **NEW**: Detailed zoom-in on Control Unit responsibilities
- Shows 5 internal components: Event Processor, Decision Engine, Storage Manager, Auth Manager, Connectivity Manager
- 6 input ports (sensors, cameras, user commands, cloud) and 6 output ports (alarms, controllers, notifications, logs)
- Item flows showing data movement between components
- 3 critical sequences (motion detection, cloud sync, offline autonomy)
- Requirements traceability showing which components satisfy which requirements
- Design notes on testability, message-driven architecture, and event sourcing

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

---

## Wireless Coverage Verification (REQ-012)

The **Deployment-Diagram.drawio** includes detailed wireless coverage annotations demonstrating compliance with **REQ-012: Motion Sensor Wireless Range**.

### Requirement Details

| Aspect                  | Specification                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------ |
| **Requirement**         | REQ-012: Motion Sensor Wireless                                                            |
| **Specification**       | Motion sensors shall communicate wirelessly to hub with range ≥ 100 meters (line of sight) |
| **Acceptance Criteria** | Successfully transmit events from corner rooms to hub; < 5% packet loss                    |
| **Technology**          | WiFi/LoRaWAN mesh network                                                                  |

### Coverage Analysis

**Deployment Scenario** (6-zone home typical layout):

- Hub Location: Center of home
- Zones: Entry, Living Room, Back Door, Master Bedroom, Kitchen, Garage

The 100m range requirement is designed to:

1. **Handle large homes**: Even 50m diagonal homes have 50m safety margin
2. **Support WiFi mesh**: Multiple APs extend effective range via relay
3. **Ensure reliability**: Link budget handles obstacles (walls, interference)
4. **Enable outdoor coverage**: Patio, deck, and exterior sensor placement

---

## Control Unit Internal Block Diagram (IBD) - Hierarchical Decomposition

The **Control-Unit-IBD.drawio** provides a detailed zoom-in on the central hub's internal architecture, showing how the Control Unit satisfies its system-level responsibilities through five integrated subsystems.

### Hierarchy and Abstraction Levels

| Level              | Artifact                          | Content                                 | Audience                |
| ------------------ | --------------------------------- | --------------------------------------- | ----------------------- |
| **System**         | System Context (Mermaid in SysRS) | 5 subsystems, connectivity types        | Stakeholders, reviewers |
| **Block**          | System-Blocks.drawio              | 8 SysML blocks, required/provided ports | Systems engineers       |
| **Component**      | Control-Unit-IBD.drawio           | 5 internal parts, item flows, sequences | Detailed designers      |
| **Implementation** | Code repositories                 | Actual classes, APIs, modules           | Developers              |

### Control Unit Internal Structure

The Control Unit contains **5 core subsystems**:

#### 1. **Event Processing Engine** (eventProcessor)

- **Responsibility**: Aggregates and correlates sensor events
- **Key Functions**:
  - Deduplicate motion events (500ms window)
  - Temporal/spatial correlation (e.g., multiple sensors firing in sequence)
  - Event queue for decision logic
  - Handles 100+ events/second throughput
- **Inputs**: Motion, door, water, camera events
- **Outputs**: Processed event stream to decision engine
- **Tests**: VER-001-008 (sensor detection), VAL-022 (latency under load)

#### 2. **Local Decision Engine** (decisionEngine)

- **Responsibility**: Evaluates automation rules and makes local decisions
- **Key Functions**:
  - Interpret rule engine (JSON-based rules or Python expressions)
  - Trigger alarms independently of cloud
  - Execute failover logic (connectivity changes)
  - Enforce offline autonomy (operate without internet)
  - Handle state machine transitions
- **Inputs**: Processed events, user commands, connectivity status
- **Outputs**: Alarm signals, controller commands, notifications
- **Tests**: VER-017 (hub failsafe), VAL-008 (end-to-end workflow)

#### 3. **Local Storage Manager** (storageManager)

- **Responsibility**: Manages persistent storage and data retention
- **Key Functions**:
  - SQLite database for events (30-day rotating log)
  - Offline video buffering (on disk or attached USB)
  - Index optimization for quick retrieval
  - Event sourcing (append-only events for audit trail)
  - Sync tracking (which events sent to cloud)
- **Inputs**: Event streams, video frames, system diagnostics
- **Outputs**: Event history, offline autonomy capability
- **Tests**: VER-029 (local storage), VAL-016-017 (off-grid operation)

#### 4. **Authentication Manager** (authManager)

- **Responsibility**: Enforces user security and permissions
- **Key Functions**:
  - Local login (username/password with bcrypt)
  - Session management (tokens, expiry)
  - Role-based access control (admin, user, guest)
  - Permission enforcement per device/action
  - Audit logging (who did what, when)
  - Optional integration with cloud IAM (SAML/OAuth)
- **Inputs**: User credentials, access requests, commands
- **Outputs**: Authenticated sessions, permission grants/denials
- **Tests**: VER-021-026 (authentication variants), VAL-019 (MFA flow)

#### 5. **Connectivity Manager** (connectivityMgr)

- **Responsibility**: Manages multiple transport protocols and failover
- **Key Functions**:
  - Monitor WiFi signal strength and availability
  - Fallback to cellular when WiFi drops
  - Route through mesh network if available
  - Failover to satellite as last resort
  - Schedule cloud sync (every 5 minutes or on-demand)
  - Abstract protocol differences (MQTT, CoAP, REST, proprietary)
  - Buffer cloud messages during outages
- **Inputs**: Network status, cloud sync requests, sensor events
- **Outputs**: Cloud connection status, outgoing messages, diagnostics
- **Tests**: VER-027-028 (WiFi/cellular failover), VAL-015-018 (mesh/satellite)

### Data Flows (Item Flows)

**Path 1: Real-Time Event Processing**

```
motionEvents → eventProcessor.aggregate()
  → deduplicate() [<500ms]
  → correlation() [temporal/spatial]
  → processedEvents → decisionEngine.evaluate()
  → rule match?
    → YES: trigger() → alarmTriggers (& notifications & control commands)
    → NO: ignore()
  → ALL: logStream → storageManager.queue()
Latency: <500ms total (local only, no cloud roundtrip)
```

**Path 2: Cloud Synchronization**

```
[periodic, every 5 minutes]
connectivityMgr.isConnected()
  → YES: storageManager.getUnsynced()
    → filter(last 4 hours)
    → connectivityMgr.send()
    → cloudPlatform receives eventLog
    → storageManager.markSynced()
  → NO: queue locally (will retry on reconnection)
```

**Path 3: Offline Autonomy**

```
connectivityMgr detects network loss
  → decisionEngine.mode = LOCAL_ONLY
  → ALL processing continues (events, rules, storage)
  → NO cloud notifications (stored locally)
  → NO outbound messages (queued)
  → On reconnect: sync queued eventLog
  → User sees "Offline Mode" indicator on app (once reconnected)
```

### Ports and Interfaces

**Boundary Inputs** (6 ports):

- `motionEvents`: EventStream from motion sensors (event objects with timestamp, location, confidence)
- `doorEvents`: EventStream from door/window sensors
- `waterEvents`: EventStream from water leak sensors
- `videoFeeds`: VideoStream from cameras (optional, for recording triggers)
- `userCommands`: Command stream (arm, disarm, query, control devices)
- `cloudSync`: Periodic synchronization data from cloud (config updates, remote commands)

**Boundary Outputs** (6 ports):

- `alarmTriggers`: AlarmSignal to siren, strobe, or notification system
- `controllerCommands`: Commands to smart locks, lights, outlets
- `notifications`: Alerts/messages to user (app, SMS, push)
- `videoRecord`: Signal to start/stop camera recording
- `eventLog`: Periodic sync of events to cloud backend
- `diagnostics`: System status (uptime, memory, connectivity, battery)

### Requirements Traceability

Each component maps to specific requirement clusters and tests:

| Component           | Requirements                                                 | Tests                    | Acceptance Criteria                                    |
| ------------------- | ------------------------------------------------------------ | ------------------------ | ------------------------------------------------------ |
| **eventProcessor**  | REQ-010-019 (Motion/Door/Water detection)                    | VER-001-008, VAL-022     | <500ms latency, <5% false positives                    |
| **decisionEngine**  | REQ-050-070 (Automation), REQ-281-285 (State machines)       | VER-017, VAL-008         | Rules evaluated locally, offline operation works       |
| **storageManager**  | REQ-180-190 (Local storage), REQ-270-280 (Offline operation) | VER-029, VAL-016-017     | 30-day log maintained, 4+ hr autonomy                  |
| **authManager**     | REQ-100-110 (Security), REQ-140 (RBAC)                       | VER-021-026, VAL-019     | Login/logout works, permissions enforced, MFA optional |
| **connectivityMgr** | REQ-120-125 (Failover), REQ-290-300 (Off-grid modes)         | VER-027-028, VAL-015-018 | Seamless WiFi↔Cellular switch, mesh/sat optional       |

### Design Principles

1. **Offline-First**: All core functionality works without cloud; cloud is enhancement
2. **Event Sourcing**: Events immutable and append-only for audit trail and replay
3. **Message-Driven**: Async, loosely coupled components enable resilience
4. **Stateless Flows**: Events contain full context (idempotent replay)
5. **Testable**: Each component is independently testable (unit test compatible)
6. **Observable**: Diagnostics port provides visibility into hub health
