# IoT Behavioral Intrusion Detection System

A real-time behavioral fingerprinting system for detecting cloned and compromised IoT devices using machine learning and cryptographic audit logging.

---

## Screenshots

### Security Dashboard
![Dashboard](public/PhantomGuard_1.png)

### Dashboard Details
![Dashboard Detail](public/PhantomGuard_2.png)

### InfluxDB Telemetry
![InfluxDB](public/influx-dashboard_1.png)

### Hyperledger Explorer
![Hyperledger Explorer](public/explorer.png)

---

## Overview

Traditional IoT security relies on device identity verification (UID, firmware hash), which can be defeated by cloning. This system uses **behavioral fingerprinting** to detect suspicious devices through:

- **Sensor patterns** — Temperature and humidity deviation analysis
- **Timing patterns** — Packet transmission interval consistency
- **Statistical anomalies** — Deviation from learned device behavior

Even with identical credentials, attackers may exhibit different physical and behavioral patterns from legitimate devices.

---

## Key Features

- **Behavioral Detection** — Identifies suspicious devices through timing and sensor analysis.
- **Multi-Layer Pipeline** — Rule-based filters, ML models, and automatic quarantine.
- **Real-Time Dashboard** — React UI with live violation tracking and device management.
- **Cryptographic Audit** — SHA-256 Merkle tree for tamper-evident logging.
- **Blockchain Integration** — Optional Hyperledger Fabric audit trail.

---

## System Architecture

    ESP32 Devices (Legitimate + Attacker)
                ↓
        Mosquitto MQTT Broker
                ↓
            FastAPI Backend
          ├─ Rule-Based Detection
          ├─ ML Inference (XGBoost, Random Forest, Isolation Forest)
          └─ Auto-Quarantine System
                ↓
        InfluxDB (Time-Series Storage)
                ↓
    Merkle Tree + Hyperledger Fabric (Audit Log)
                ↓
        React Dashboard (Visualization)

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| Accuracy | 94.2% |
| Precision | 91.7% |
| Recall | 96.3% |
| F1-Score | 0.94 |
| ROC-AUC | 0.96 |
| Inference Latency | ~180 ms |
| False Positive Rate | 8.3% |

*Evaluated on 2,000+ test samples using a SMOTE-balanced dataset.*

---

## Technology Stack

| Component | Technologies |
|-----------|--------------|
| Hardware | ESP32 × 2, DHT11 sensors |
| Communication | MQTT, Mosquitto |
| Backend | Python 3.9+, FastAPI, Uvicorn |
| Database | InfluxDB 2.x |
| Machine Learning | XGBoost, scikit-learn, SHAP |
| Frontend | React 19, Recharts |
| Blockchain | Hyperledger Fabric (optional) |

---

## Detection Logic

### Rule-Based Layer

- **TAMPER** — Firmware hash mismatch.
- **ANOMALY** — Temperature above 40°C or below 15°C; humidity below 25% or above 80%.
- **Behavioral Deviation** — Transmission interval below 3,000 ms.

### Machine Learning Layer

Three models analyze behavioral features:

- `temperature`
- `humidity`
- `interval`
- `temp_deviation` — Distance from the 30°C baseline.
- `interval_deviation` — Distance from the 5,000 ms baseline.

### Automatic Quarantine

Devices with three or more violations are blocked for five minutes.

---

## Project Structure

    backend/
    ├── telemetry_api_v3.py        # FastAPI, MQTT, and detection
    ├── train_models_v3_FIXED.py   # ML training pipeline
    ├── merkle_logger.py           # Cryptographic audit logging
    ├── hlf_client.py              # Blockchain client
    └── ids-chaincode.js           # Hyperledger Fabric smart contract

    hardware/
    ├── ESP_Legit/                 # Legitimate device firmware
    └── ESP_Ghost/                 # Attack simulator firmware

    dashboard/
    └── src/
        ├── Dashboard.js           # Main React component
        └── Dashboard.css

    docs/
    ├── EXECUTION_GUIDE_30MIN.md   # Setup guide
    └── visualizations/            # ML performance charts

---

## Running the Project

For installation, configuration, and execution instructions, see:

- [Execution Guide](docs/EXECUTION_GUIDE_30MIN.md)
- [Dashboard Setup](docs/REACT_DASHBOARD_SETUP.md)

### Prerequisites

- Python 3.9+
- Node.js 18+
- Mosquitto MQTT broker
- InfluxDB 2.x
- Arduino IDE (for ESP32 hardware)

### Backend Setup

    cd backend
    pip install -r requirements.txt

### Dashboard Setup

    cd dashboard
    npm install

### Start the Services

Run each service in a separate terminal.

**Terminal 1 — MQTT Broker**

    mosquitto -v

**Terminal 2 — InfluxDB**

    influxd

**Terminal 3 — Backend API**

    cd backend
    python telemetry_api_v3.py

**Terminal 4 — Dashboard**

    cd dashboard
    npm start

Open `http://localhost:3000` to access the dashboard.

Configure your InfluxDB organization, bucket, and API token before starting the backend.

---

## ESP32 Setup

### Hardware

- Two ESP32 boards
- Two DHT11 temperature and humidity sensors
- Connect each DHT11 sensor to GPIO 4 on its respective ESP32.

### Firmware

1. Open `hardware/ESP_Legit/ESP_Legit.ino` in Arduino IDE.
2. Configure Wi-Fi credentials and the MQTT server IP.
3. Flash the firmware onto the legitimate device.
4. Repeat the process for `hardware/ESP_Ghost/ESP_Ghost.ino` on the second device.

The Ghost device simulates attack scenarios, including behavioral cloning, firmware tampering, sensor anomalies, and combined attacks.

---

## Dashboard Features

- **Live Metrics** — Devices tracked, violations, and audit entries.
- **Violation Charts** — Real-time time-series visualization.
- **Merkle Verification** — Cryptographic audit-chain validation.
- **Device Management** — Per-device quarantine controls.
- **Recent Events** — Latest security alerts with timestamps.

---

## Optional: Hyperledger Fabric

Deploy the blockchain audit log in WSL/Linux using the Fabric test network:

    cd ~/fabric-samples/test-network
    ./network.sh up createChannel -c mychannel -ca
    ./network.sh deployCC -ccn idsaudit -ccp "/path/to/backend" -ccl javascript

Hyperledger Explorer is available at `http://localhost:8080` when configured and running.

---

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/stats` | GET | System statistics |
| `/telemetry` | POST | Submit telemetry data |
| `/verify-logs` | GET | Verify the Merkle audit chain |
| `/recent-events` | GET | Retrieve recent security events |
| `/quarantine/{uid}/release` | POST | Release a quarantined device |

API documentation: `http://localhost:8000/docs`

---

## Documentation

- [Execution Guide](docs/EXECUTION_GUIDE_30MIN.md) — Complete setup instructions.
- [Dashboard Setup](docs/REACT_DASHBOARD_SETUP.md) — Frontend configuration.

---

## Authors

**Batch 13 — IoT (Year 3, Semester 2) — MVSREC**

| Roll No | Name |
|---------|------|
| 2451-23-749-040 | Chavada Shashank |
| 2451-23-749-051 | P Samuel Vijay |
| 2451-23-749-060 | Arikela Sriharsh |

---

## License

Academic project — MVSREC.

---

## Acknowledgments

Built with FastAPI, React, InfluxDB, Hyperledger Fabric, and SHAP explainability.
