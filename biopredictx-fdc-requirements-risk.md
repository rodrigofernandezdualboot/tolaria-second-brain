---
type: Note
_display: sheet
belongs_to: "[[biopredictx-fdc-alpha-2-0]]"
related_to:
  - "[[biopredictx-alpha-2-0]]"
  - "[[biopredictx-scope-requirements-comparison]]"
status: Active
_sheet:
  frozen_rows: 1
  columns:
    A:
      width: 128.01171875
    B:
      width: 460
    C:
      width: 80
    D:
      width: 480
---
,,,
ID,Definition (as stated),Risk,Comments
FR-A1,Operations Portal — User Configuration: register new test user; assign Restiv device.,Low,Standard CRUD web UI; no hardware or real-time dependency.
FR-A2,Operations Portal — Device Configuration: set Restiv therapy settings — magnet rotation angles; motor acceleration; rotational frequency.,Medium,Depends on the existing Restiv firmware exposing these parameters over the RPi–Arduino interface; if not exposed the interface must be defined or extended.
FR-A3,Operations Portal — Device Status: device list; network status.,Low,Conventional status display over device metadata.
FR-A4,Operations Portal — Data Retrieval: wearable raw-data reports; sleep-stage timestamps; therapy reports — start; stop; interrupted.,Low,Reporting over stored data; depends only on the Data Storage Layer being in place.
FR-A5,API & Services Layer — standardized interfaces across UIs; backend; analytics; devices (auth; authorization; device commands; data access; session management).,Medium,Conventional API backbone but must carry real-time device commands and session state; foundational so defects propagate widely.
FR-A6,Data Acquisition Layer — acquire biometric/telemetry over Bluetooth LE; validation; time-stamping; buffering; streaming.,High,Real-time streaming from a consumer wearable is unproven here; Garmin-first implies a Garmin Connect cloud path that conflicts with real-time acquisition. Central open dependency.
FR-A7,Analytics Layer — signal filtering; feature extraction; sleep-stage determination; outputs six timestamped stages; COTS/Garmin-first staging.,High,COTS/Garmin staging may not surface emerging-deep-sleep in real time; the FDC open item admits custom software may be required.
FR-A8,Therapy Control Layer — device command transmission; status monitoring; therapy ON/OFF logic.,High,ON logic must satisfy the pre-deep-sleep timing constraint (NFR-A2); the command path depends on the existing firmware accepting live commands at low latency.
FR-A9,Data Storage Layer — persist raw sensor data; processed data; sleep-stage outputs; therapy on/off logs; device status logs.,Low,Conventional tiered storage; hot/warm/cold model already sketched.
FR-A10,Access control — internal vs external user permissions matrix; de-identified User Record and Device Record.,Low,Standard role-based access; de-identification is lightweight (a de-identified User ID).
FR-A11,Operational state machine — Initialization; Idle; Device Pairing; Ready; Monitoring; Therapy Active; Session Complete; Fault; Recovery; sleep-stage loop repeats 3–6× per session.,Medium,Well specified but must coordinate real-time transitions; fault-from-any-state and safe therapy shutdown add complexity.
,,,
Non-Functional Requirements,,,
NFR-A1,Real-time closed-loop operation — system operates as a real-time virtual machine; stream biometrics to detect emerging deep sleep.,High,The defining constraint of Alpha 2.0; every time-critical behavior depends on it.
NFR-A2,Therapy timing constraint — therapy must activate before the user enters deep sleep; prioritize an early estimate over a later higher-confidence one.,High,Hardest requirement in the phase; the FDC flags that COTS staging may fail to meet it and force custom development.
NFR-A3,COTS-first / API-first — use COTS to the greatest extent; minimize custom software.,Medium,In direct tension with NFR-A2; if COTS cannot hit the timing target the minimize-custom goal breaks.
NFR-A4,Administrative controls over engineered controls.,Low,Deliberately reduces engineering burden; lowers overall build risk.
NFR-A5,Extensible modular design — build for source/module reuse and a clean Beta transition.,Low,Design discipline; minor risk of premature abstraction.
NFR-A6,De-identification — maintain a unique de-identified User ID per participant.,Low,Lightweight compared with the old scope's engineered privacy stack.
NFR-A7,Fault tolerance / safe state — Fault reachable from any state; preserve data; safe state; disable therapy; then Recovery.,Medium,Safety-relevant; disabling therapy cleanly on any fault needs care but is bounded.
NFR-A8,[Beta] Single frontend codebase deployable as web now and iOS/Android later.,Low,Forward-looking groundwork; not Alpha-critical.NFR-A9,[Beta] Clearly defined interface point for a future proprietary black-box algorithm.,Medium,Must define the right seam now without the algorithm existing; risk the interface guesses wrong.