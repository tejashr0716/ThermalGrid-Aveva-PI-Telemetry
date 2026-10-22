# ❄️ ThermalGrid — Hyperscale Data Center & Smart Facility Energy/Thermal Telemetry via AVEVA OSIsoft PI Web API

[![AVEVA OSIsoft PI](https://img.shields.io/badge/AVEVA-OSIsoft%20PI%20System-orange?style=flat-square&logo=aveva)](https://www.aveva.com)
[![PI Web API](https://img.shields.io/badge/PI%20Web%20API-Batch%20%26%20Channels-blue?style=flat-square)](https://docs.aveva.com)
[![Asset Framework](https://img.shields.io/badge/PI%20AF-PUE%20Formula%20Engine-purple?style=flat-square)](https://docs.aveva.com)
[![FastAPI](https://img.shields.io/badge/FastAPI-Sub--second%20Telemetry-emerald?style=flat-square)](https://fastapi.tiangolo.com)
[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-success?style=flat-square)](https://tejashr0716.github.io/thermalgrid/)

> **Enterprise Custom Software Engineering Solution for Accenture Industry X & Smart Infrastructure Practice**  
> Real-time industrial thermal and power usage telemetry engine integrating AVEVA OSIsoft PI Data Archive, PI Asset Framework (AF), and PI Web API for a multi-megawatt AI GPU data center.

---

## 📌 Executive Summary & Accenture Alignment

Modern hyperscale AI data centers draw over $100	ext{ kW}$ per rack, creating thermal runaway risks within seconds of a chiller valve disruption. Maintaining a Power Usage Effectiveness ($	ext{PUE} < 1.15$) requires continuous sub-second monitoring of liquid cooling loops, server rack inlet temperatures, and ASHRAE TC9.9 thermal envelopes.

**ThermalGrid** bridges **AVEVA OSIsoft PI** operational historian data with modern cloud-native web architectures:
- **High-Density Sensor Ingestion:** Ingests over 5,000 thermal and power tags via PI Web API batch endpoints (`POST /piwebapi/batch`) and WebSocket channels.
- **Hierarchical PUE Modeling:** Automatically calculates dynamic facility PUE using PI Asset Framework (AF) formula references:
  $$	ext{PUE} = rac{	ext{Total Facility Power (kW)}}{	ext{Total IT Power (kW)}}$$
- **ASHRAE Excursion Event Frames:** Automatically flags thermal hotspot runaway and chiller pump cavitation using PI AF Event Frame analyses.

---

## 🏗️ System Architecture

```mermaid
flowchart TD
    subgraph OT_DataCenter ["Data Center Field Layer (OT)"]
        Sensors["32 High-Density Racks + Chiller Plant + CDUs"]
        PowerMeters["Schneider ION Power Meters + Yokogawa Temp Transmitters"]
        Sensors --> PowerMeters
    end

    subgraph PI_System ["AVEVA OSIsoft PI System"]
        PowerMeters -->|"Modbus / BACnet / SNMP"| PI_Connector["AVEVA PI Connector"]
        PI_Connector --> PI_DA["PI Data Archive (Time-Series Tags)"]
        PI_DA <--> PI_AF["PI Asset Framework (AF Hierarchy & Formulas)"]
        PI_AF --> PI_EF["PI Event Frames (Thermal Runaway & PUE Excursions)"]
        PI_AF <--> PI_WebAPI["AVEVA PI Web API (REST & Batch Endpoints)"]
    end

    subgraph Custom_Software ["Custom Telemetry Software Layer"]
        PI_WebAPI -->|"POST /piwebapi/batch"| BatchEngine["FastAPI Batch Ingestor"]
        PI_WebAPI -->|"WSS /piwebapi/channels"| StreamHub["Async WebSocket Broadcaster"]
        BatchEngine --> RingBuffer["Zero-Allocation Typed Memory Buffer"]
        StreamHub --> RingBuffer
    end

    subgraph UI_Layer ["SCADA-Grade Client Dashboard"]
        RingBuffer --> Heatmap["2D/Canvas Server Hall Isotherm Heatmap"]
        RingBuffer --> PUEGauge["Real-Time Dynamic PUE Efficiency Gauge"]
        RingBuffer --> ChillerLoop["Liquid Cooling Direct-to-Chip Flow Diagram"]
        RingBuffer --> AFExplorer["Navigable PI AF Element Tree Explorer"]
    end
```

---

## 🌲 PI Asset Framework (AF) Hierarchy & Data Model

```
Global Cloud Hyperscale Corp (Database)
└── Bengaluru DC1 - Whitefield Campus (Site Element)
    ├── Substation 11kV/415V
    ├── Central Chiller Plant
    │   ├── Chiller-01 (Centrifugal Chiller 1,200 TR)
    │   ├── Chiller-02 (Centrifugal Chiller 1,200 TR)
    │   ├── Cooling Towers (CT-01 to CT-04)
    │   └── CDUs (Coolant Distribution Units 1 & 2)
    └── Server Hall 01
        ├── Row A (AI Training Clusters - Racks A01 to A08)
        ├── Row B (Compute Nodes - Racks B01 to B08)
        ├── Row C (Storage Arrays - Racks C01 to C08)
        └── Row D (Spine Network - Racks D01 to D08)
```

### Key AF Attributes & Formulas:

| Attribute Name | Data Reference | Description | Engineering Unit |
|---|---|---|---|
| `Total_Facility_Power` | PI Point (`DC1.Substation.ActivePower`) | Gross electrical intake from grid + solar | $	ext{kW}$ |
| `Total_IT_Power` | PI Point (`DC1.PDU.TotalITLoad`) | Net electrical power consumed by server racks | $	ext{kW}$ |
| `PUE_Ratio` | Formula Reference | `Total_Facility_Power / Total_IT_Power` | Dimensionless |
| `Supply_Chilled_Water_Temp` | PI Point (`DC1.Chiller.SupplyTemp`) | Primary chilled water leaving evaporator | $^\circ	ext{C}$ |
| `Return_Chilled_Water_Temp` | PI Point (`DC1.Chiller.ReturnTemp`) | Water returning from air handlers & CDUs | $^\circ	ext{C}$ |
| `Primary_Loop_Flow` | PI Point (`DC1.CDU.PrimaryFlow`) | Direct-to-chip coolant circulation rate | $	ext{LPM}$ |
| `Rack_Inlet_Air_Temp` | PI Point (`DC1.RackA01.InletTemp`) | Front-door cold aisle air temperature | $^\circ	ext{C}$ |
| `Hot_Aisle_Delta_T` | Formula Reference | `Rack_Exhaust_Temp - Rack_Inlet_Temp` | $^\circ	ext{C}$ |

---

## ⚡ Key PI Web API Endpoints Implemented

1. **High-Speed Batch Retrieval:**
   * `POST /piwebapi/batch` — Queries all 32 rack temperature and power attributes in 1 single HTTP request, cutting network overhead by $96\%$.
2. **Dynamic PUE Formula Calculation:**
   * `GET /piwebapi/streamsets/getinterpolated` — Retrieves synchronized facility power and IT load to compute rolling PUE trends.
3. **Automated Event Frame Generation:**
   * `POST /piwebapi/eventframes` — Creates PI Event Frames for ASHRAE TC9.9 temperature violations ($>27^\circ	ext{C}$) and PUE spikes ($>1.25$).

---

## 🎯 Accenture Technical Interview Guide (Cheat Sheet)

### Q1: How does PI Web API handle authentication in enterprise environments?
> **Answer:** PI Web API supports Kerberos/Negotiate (standard for Active Directory on-premise), Basic Auth (for TLS edge relays), and modern OAuth2 / OpenID Connect (OIDC) via Azure AD/Entra ID. In cloud-hosted architectures, we configure PI Web API behind an API Gateway with Bearer token authentication mapping claims to PI AF security identities.

### Q2: What is the OSIsoft Message Format (OMF) and when should it be used with PI Web API?
> **Answer:** OMF is a JSON-based specification for sending high-throughput time-series data, asset types, and element definitions into the PI System via PI Web API (`/piwebapi/omf`). It is designed for edge IoT gateways (like Raspberry Pi, Moxa, or Siemens edge devices) because it decouples device code from internal PI WebIDs, automatically generating AF elements and PI Points upon first data delivery.

---

## 🚀 Live Demo & Quick Start

* **Live Interactive Application:** [https://tejashr0716.github.io/thermalgrid/](https://tejashr0716.github.io/thermalgrid/)
* **Local Run:**
  ```bash
  git clone https://github.com/tejashr0716/ThermalGrid-Aveva-PI-Telemetry.git
  cd ThermalGrid-Aveva-PI-Telemetry
  pip install -r requirements.txt
  python src/main.py
  ```
