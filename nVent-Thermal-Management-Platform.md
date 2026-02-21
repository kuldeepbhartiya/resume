# nVent Thermal Management Platform

## 1. Summary
The nVent Thermal Management Platform is a real-time industrial IoT solution for monitoring and controlling heating systems, including heating cables, controllers, thermostats, and HMI panels. The system collects telemetry from controllers via Modbus TCP/RTU, processes it in Golang microservices, and provides real-time operator dashboards through Angular-based HMI panels.

The platform is designed for industrial, commercial, and residential applications, including pipe freeze protection, roof/gutter de-icing, tank heating, and rail switch heating. The architecture ensures low-latency updates, high availability, and persistent data storage for analytics, monitoring, and predictive maintenance readiness.

---

## 2. Business Problem
- Industrial heating systems require real-time monitoring and control to prevent downtime or safety incidents.
- Operators need a responsive UI (HMI panels) that reflects device state immediately.
- Data must be persisted for historical analysis, diagnostics, and regulatory compliance.
- Initial deployments faced startup latency issues due to hosting Angular static files via AWS Lambda.
- Multi-device, multi-protocol integration (Modbus TCP/RTU, REST API) is required for large-scale industrial environments.

---

## 3. High-Level Architecture Overview
The architecture consists of four primary layers:

### HMI Layer (UI)
- Angular frontend hosted on AWS S3 static hosting and delivered via CloudFront for low-latency access.
- Displays dashboards, device status, alarms, and control panels.
- Connects to backend via REST API or WebSockets/SSE for real-time updates.

### Backend Microservices Layer
- Golang microservices, responsible for orchestrating all data flows.
- Provides:
  - REST API endpoints for HMI panels.
  - Socket/Modbus communication to controllers.
  - Telemetry processing and persistence to MySQL database.
- Stateful session management ensures that pods reconnect to the same controllers after restarts.
- Deployed on AWS Fargate, replacing Lambda to reduce cold-start latency and maintain long-lived connections.

### Controller Interface Layer
- Connects directly to industrial controllers and devices via Modbus TCP/RTU over sockets.
- Reads sensor data (temperature, alarms, power) and writes commands (enable/disable, thresholds).
- Provides low-latency telemetry feedback to backend for HMI panels.

### Database Layer
- MySQL stores telemetry history, configuration, and operator actions.
- Supports analytics, reporting, and integration with downstream systems.

---

## 4. Component Sequence & Data Flow
```
[HMI Panel (Angular UI)]
        |
        | REST API / WebSocket updates
        v
[Backend Microservices (Golang, Fargate)]
        |
        | Socket / Modbus TCP-RTU
        v
[Controllers / Heating Devices]
        |
        | Telemetry feedback
        v
[Backend Microservices] --> [MySQL Database]
        |
        v
[HMI Panel (real-time updates)]
```

### Notes on the Flow:
- HMI sends commands → Backend → Controllers.
- Controllers send telemetry → Backend → HMI (real-time) + DB (persistent).
- In-memory caching or pub/sub used for rapid HMI updates.
- Stateful sessions maintain controller connections even if backend pods restart.

---

## 5. Technology Stack
| Layer                  | Technology / Tools                                   |
|------------------------|-----------------------------------------------------|
| **Frontend HMI**       | Angular, AWS S3, CloudFront                         |
| **Backend Microservices** | Golang, Fargate/ECS, REST APIs                   |
| **Controller Communication** | .NET / Golang APIs, Modbus TCP/RTU, Socket programming |
| **Database**           | MySQL                                              |
| **Messaging / Caching** | In-memory cache / pub-sub for low-latency updates  |
| **Load Balancing & Routing** | Nginx Ingress, AWS Application Load Balancer  |
| **Observability**      | CloudWatch, Prometheus, Grafana                    |

---

## 6. Key Design Considerations

### Real-time Performance
- Telemetry updates reflected on HMI within milliseconds.
- Backend caches data to avoid DB read latency.

### High Availability
- Backend microservices deployed on AWS Fargate across multiple availability zones (multi-AZ).
- Stateful sessions ensure pods reconnect to same controllers on restart.

### Protocol Handling
- Integrated Modbus TCP/RTU for device communication.
- Backend abstracts protocols for HMI via standard REST APIs.

### Data Persistence
- All telemetry and commands stored in MySQL for audit, analytics, and reporting.

### Startup & Scalability
- Replaced AWS Lambda serving Angular with S3 + CloudFront + Fargate to reduce cold starts.
- Backend microservices horizontally scalable via Fargate task auto-scaling.

---

## 7. Achievements & Metrics
- **Real-time processing:** 1–5k telemetry messages/sec from multiple controllers.
- **Data volume:** ~100–300 MB/day per site persisted in MySQL.
- **Response time:** <50ms update latency to HMI.
- **High availability:** Multi-AZ Fargate deployment with stateful sessions.
- **Protocol coverage:** Supported Modbus TCP/RTU and REST APIs for heterogeneous industrial devices.