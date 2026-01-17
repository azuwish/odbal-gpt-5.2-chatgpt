# OBD-II Power2 Diagnostic Scanner – Python Application Specification

## Purpose
This document defines the requirements and design for a Python application that communicates with a **Power2 OBD-II diagnostic scanner**, reads live vehicle data, and retrieves stored diagnostic trouble codes (DTCs). It is intended to be used as **input to code generation** (human or AI-assisted) and should be treated as an authoritative source of truth for functionality and constraints.

---

## Target Audience
- Python developers
- Automotive diagnostics enthusiasts
- AI/code-generation systems

---

## Goals
1. Establish reliable communication with a Power2 OBD-II scanner
2. Display **live vehicle telemetry** in near real-time
3. Read and display **stored and pending DTCs**
4. Provide a clear, extensible architecture
5. Remain portable across macOS, Linux, and Windows

---

## Non-Goals
- ECU reprogramming or flashing
- Emissions readiness certification
- Manufacturer-specific (non-OBD-II) coding

---

## Hardware Assumptions
- **Scanner**: Power2 OBD-II diagnostic scanner
- **Vehicle**: Any OBD-II–compliant vehicle (1996+ US)
- **Connection Type** (one or more):
  - USB serial
  - Bluetooth (SPP)

---

## Software Environment

### Runtime
- Python **3.10+**

### Recommended Libraries
- `python-OBD` – high-level OBD-II communication
- `pyserial` – serial communication fallback
- `bleak` – Bluetooth LE support (if required)
- `rich` or `textual` – terminal UI
- `asyncio` – non-blocking I/O

---

## High-Level Architecture

```
+------------------+
|  User Interface  |
| (CLI / TUI)      |
+---------+--------+
          |
          v
+------------------+
| Data Controller  |
| - polling        |
| - scheduling     |
+---------+--------+
          |
          v
+------------------+
| OBD Interface    |
| - connect        |
| - query PIDs     |
+---------+--------+
          |
          v
+------------------+
| Power2 Scanner   |
| (Vehicle ECU)    |
+------------------+
```

---

## Application Modes

### 1. Live Data Mode
Continuously polls selected OBD-II PIDs and displays them.

**Polling interval:** configurable (default: 1 second)

### 2. Diagnostic Mode
Reads and displays:
- Stored DTCs
- Pending DTCs
- MIL (Check Engine) status

---

## Supported OBD-II Data (Initial Scope)

### Core Live PIDs
| PID | Description |
|----|------------|
| 010C | Engine RPM |
| 010D | Vehicle Speed |
| 0105 | Coolant Temperature |
| 010B | Intake Manifold Pressure |
| 0111 | Throttle Position |
| 0104 | Engine Load |
| 0110 | Mass Air Flow |


### Diagnostic Queries
- `03` – Stored DTCs
- `07` – Pending DTCs
- `01 01` – MIL status

---

## User Interface Requirements

### CLI / TUI Features
- Real-time updating dashboard
- Clear separation between:
  - Live data
  - Diagnostic codes
- Color-coded warnings (e.g., MIL active)

### Example Layout (Textual)
```
RPM:        850        Coolant:   92°C
Speed:      0 mph      Throttle:  14%
MAF:        2.1 g/s    Load:      18%

MIL: OFF
Stored DTCs: None
```

---

## Configuration

### Configuration File (YAML or TOML)
```yaml
connection:
  type: auto        # auto | serial | bluetooth
  port: null        # e.g. /dev/ttyUSB0
  baudrate: 38400

polling:
  interval_ms: 1000

pids:
  - RPM
  - SPEED
  - COOLANT_TEMP
```

---

## Error Handling

### Connection Errors
- Scanner not found
- Permission denied (serial port)

### Runtime Errors
- Unsupported PID
- Timeout or malformed response

**Requirement:**
- Errors must be user-readable
- Application must continue running when possible

---

## Logging

- Use Python `logging` module
- Log levels:
  - INFO – connection status
  - WARNING – unsupported PIDs
  - ERROR – communication failure

Optional file logging:
```
~/.power2_obd/logs/app.log
```

---

## Extensibility Considerations

- PID list must be declarative
- UI layer must not directly access OBD interface
- Support future features:
  - Freeze-frame data
  - CSV export
  - Graphing

---

## Security & Safety

- Read-only access to vehicle ECU
- No write commands allowed
- Defensive parsing of all scanner input

---

## Testing Strategy

### Unit Tests
- PID parsing
- DTC decoding

### Integration Tests
- Mock OBD adapter
- Simulated ECU responses

---

## Deliverables

1. Python application source code
2. Configuration template
3. README for end users
4. This specification document

---

## Success Criteria
- Application connects to Power2 scanner without manual tuning
- Live data updates smoothly
- Stored and pending codes are displayed accurately
- Codebase is readable, modular, and testable

---

## Notes for Code Generation Systems

- Prefer `python-OBD` abstractions where available
- Fall back to raw PID queries only if required
- Avoid blocking calls in UI loop

---

**End of Specification**

