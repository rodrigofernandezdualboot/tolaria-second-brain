---
type: Note
belongs_to: "[[biopredictx-alpha-2-0]]"
status: Active
---

# BioPredictX FDC — Alpha 2.0 (New Scope)

Functional Design Criteria for the Restiv system, Alpha 2.0 Software Development. Draft, 7/1/26, author Lindsey Wilkinson. This is the **new, constrained clinical-feasibility scope** that replaces [[biopredictx-scope-v2]]. See [[biopredictx-alpha-2-0]] for the comparison.

## Purpose & intended use

Defines functional criteria for the Restiv system — software, cloud services, IoT comms, data pipelines, and therapy control. Supports the Alpha 2.0 feasibility study to verify and validate the Restiv prototype at basic functionality for a minimal-use Alpha test regime. Consumer-facing features are out of scope for this phase.

## Product overview & philosophy

Integrate a newly built backend framework, the existing Restiv device firmware, and a wearable health-sensor interface. System operates as a **real-time virtual machine** in a cloud-based architecture: a Restiv device as a local machine near the user, a patient data pipeline, and a data-analytics backend.

Development philosophy, to the greatest extent possible:

- **COTS-first with API-first strategy** — identify and use commercial off-the-shelf technologies.
- **Administrative controls over engineered controls** — lean on manual process instead of complex automation.
- **Extensible, modular design** — build for source/module reuse and a clean transition into Beta.

## System overview (hardware)

Local gateway is a **Raspberry Pi 4** (Bluetooth/Wifi) running the application server, talking to a **microcontroller** (Arduino-based) and the wellness device. Data flows up to a cloud patient-data pipeline (AWS or equal) with hot/warm/cold storage tiers. An Admin app (internal) sits on top; a User app (external) is groundwork only for Beta.

## Stakeholders & permissions

**Internal user (Administrator/Operator)** — BioPredictX staff: create/manage test-user records, register/assign/configure wellness devices, review and export data, manage settings, review logs/alerts, configure Restiv therapy parameters. Cannot start/stop sessions or generate physiological data.

**External user (Test User)** — study participant: connect Restiv to wifi, pair wearable sensor, wear sensors and operate the device per instructions. Can start/stop sessions and generate physiological/device data. Generates therapy-usage data, session metadata, and wearable health data.

System assets: de-identified User Record (User ID, assigned devices, session/physiological/therapy history) and Device Record (Device ID, device version, firmware version).

## Functional decomposition

- **Operations Portal (Web UI)** — internal-use web interface, no dev expertise required. User config (register test user, assign Restiv), device config (**magnet rotation angles, motor acceleration, rotational frequency**), device status/network, data retrieval (raw reports, sleep-stage timestamps, therapy reports: start/stop/interrupted).
- **API & Services Layer** — standard interfaces between UIs, backend, analytics, and devices.
- **Data Acquisition Layer** — acquire biometric/telemetry from wearables (Bluetooth LE), validate, time-stamp, buffer, stream.
- **Analytics Layer** — signal filtering, feature extraction, sleep-stage determination. Outputs timestamped stages: Awake, Light Sleep, **Emerging Deep Sleep**, Deep Sleep, REM, Unknown. Sleep staging leverages open-source/COTS; **Garmin SDK / Garmin Connect tried first** (strong correlation to Muse EEG, clinically tested efficacy).
- **Therapy Control Layer** — command generation, device comms, status monitoring, ON/OFF logic.
- **Data Storage Layer** — raw sensor data, processed data, sleep-stage outputs, therapy on/off logs, device status logs.

## Therapy activation logic

Activation/deactivation triggered by estimated transitions into and out of deep sleep. The "on" signal aligns with the first detected transition into deep sleep (emerging deep sleep), **prioritizing an early estimate over a later, higher-confidence one — therapy must activate BEFORE the user enters deep sleep.** "Off" signals are less critical and can come directly from standard COTS sleep-stage outputs.

## Operational state machine

Initialization → Idle → Device Pairing → Ready → Monitoring → (Therapy Active ⇄ Monitoring, sleep-stage loop repeats 3–6× per session) → Session Complete → Idle. A **Fault** state is reachable from any state (sensor/device/database/comms/API failure) and routes through **Recovery** back to Idle, or back to Fault on recovery failure. Fault preserves collected data, places the system in a safe state, and disables therapy if required.

## Beta expansion strategy

Frontend framework should support one codebase deployable as web now and iOS/Android later. Analytics and therapy-control layers should support a Beta transition from COTS sleep staging to a proprietary predictive "black box" algorithm, with a clearly defined interface point for it to plug in.

## Open items & assumptions

- **Wearable selection** — risk of change. Muse EEG has all needed sensors in one device; Garmin lacks the EEG needed for continued deep-sleep testing. Assumption: Alpha 2.0 integrates with Garmin software to support the analytics layer; Muse EEG may be used external to the system defined here.
- **Therapy control logic** — assumed that "on"-signal logic can be derived from simple alterations to standard COTS sleep staging. Risk that this fails to time therapy *prior* to deep-sleep onset, forcing custom software development in this phase.

*Source: `Functional Design Criteria (FDC) Alpha 2.0.pdf` (14 pp., BioPredicTx confidential).*
