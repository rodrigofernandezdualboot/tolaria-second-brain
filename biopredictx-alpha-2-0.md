---
type: Project
---
# BioPredictx Alpha 2.0

The new documents and diagrams for the **Alpha 2.0 Feasibility Study** represent a significant pivot in strategy. While the previous documentation laid out a highly scalable, production-ready, consumer-facing cloud architecture on AWS, the new Alpha 2.0 approach is a **highly constrained clinical testing setup** designed to verify the prototype with minimal custom software development. 

The primary differences between this newly defined approach and your previous project documentation lie in five core areas:

### 1. Shift from Batch Ingestion to Real-Time Closed-Loop Control
*   **Previous Approach:** The cloud platform was designed for a **nightly bulk upload** of sleep data (telemetry files up to 1MB) from a large device fleet, aiming to process this data retrospectively to update future treatment protocols. 
*   **Alpha 2.0 Approach:** The system operates as a **real-time closed-loop virtual machine**. Biometric data from wearables must be streamed in real-time to detect "emerging deep sleep" and instantly trigger the therapy device **before** the user enters deep sleep. 

### 2. Immediate Hardware Setup: The Gateway & Dual-Sensor Model
*   **Previous Approach:** The hardware edge layer was abstract, with an emphasis on avoiding physical hardware or firmware engineering within the cloud SOW. 
*   **Alpha 2.0 Approach:** The architecture introduces a highly specific local hardware gateway—a **Raspberry Pi 4** (with Bluetooth/Wifi) acting as an application server, communicating with an **Arduino-based Microcontroller** and the wellness device. It also leverages a dual-sensor model: a **Garmin VivoActive 5** to actively drive the real-time therapy protocol and a **MUSE EEG headband** to measure and validate the clinical sleep results. 

### 3. COTS & Open-Source Sleep Staging vs. Proprietary AI
*   **Previous Approach:** The plan heavily prioritized building custom machine learning models inside an isolated **Amazon SageMaker** private VPC, utilizing a detailed Feature Store to calculate moving averages and track strict regulatory data lineage.
*   **Alpha 2.0 Approach:** To minimize custom software development, the team will leverage **Commercial Off-The-Shelf (COTS)** or open-source sleep-staging algorithms—specifically prioritizing the **Garmin SDK/Garmin Connect** first. Custom proprietary "black box" ML algorithms are pushed out of scope to the future Beta phase.

### 4. Admin Web Portal vs. Native Mobile Apps
*   **Previous Approach:** The engineering plan focused heavily on building native-performing, cross-platform JavaScript mobile apps (iOS and Android) using a GraphQL API (AWS AppSync) and Cognito. 
*   **Alpha 2.0 Approach:** Consumer-facing mobile apps are completely out of scope. Instead, the system will feature a web-based **Operations Portal (Web UI)** for internal researchers and administrators to manage test user records, configure device parameters (like magnet rotation angles and frequencies), and export raw data. A single codebase (likely utilizing **Flutter**) will be used to lay the groundwork for a transition to mobile in the future Beta phase.

### 5. Tactical Philosophy: "Administrative over Engineered Controls"
*   **Previous Approach:** The core needs focused heavily on building automated, enterprise-grade cloud pipelines: automated AWS Glue de-identification ETLs, DynamoDB PII secure vaults, and AWS CloudTrail audit logs. 
*   **Alpha 2.0 Approach:** The development philosophy explicitly places **administrative controls over engineered controls**. Since the system is constrained to a basic, minimal-use test regime, the team can rely on manual administrative processes to handle tasks that would otherwise require highly complex, automated software engineering.

### Summary Comparison

| Architectural Dimension | Previous Production Focus | New Alpha 2.0 Focus |
| :--- | :--- | :--- |
| **Primary Goal** | Commercial D2C launch & 500k device scale | Constrained clinical feasibility testing |
| **Data Flow** | Automated nightly batch uploads over S3 | Real-time sensor streaming & active session loops |
| **Trigger Mechanism** | Retrospective analysis & future treatment adjustments | Real-time therapy activation before deep sleep |
| **User Interface** | Patient-facing iOS/Android apps (GraphQL/Cognito) | Internal Web Portal for researcher administration |
| **Algorithm Layer** | SageMaker/Bedrock proprietary ML pipelines | Garmin Connect SDK / COTS sleep staging |
| **Edge Hardware** | Abstracted under-bed device | Raspberry Pi 4 + Arduino micro + Garmin & Muse |