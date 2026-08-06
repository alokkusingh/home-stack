# Kubernetes Application Layers & Dependencies

## Application Stack Diagram

```mermaid
graph TB
    subgraph Tier1["Presentation Tier"]
        DASHBOARD["Dashboard Service<br/>Web UI<br/>http://jgte<br/>http://hdash.alok.world"]
    end

    subgraph Tier2["API/Gateway Tier"]
        subgraph APIGW["Ingress Layer"]
            INGRESS["NGINX Ingress Controller<br/>3 Ingress Rules<br/>Path-based routing"]
        end
    end

    subgraph Tier3["Authentication Tier"]
        AUTH["home-auth-service:8081<br/>🔒 Security<br/>- OAuth 2.0 (Google)<br/>- JWT Token Management<br/>- Session Management<br/>- User Authentication<br/><br/>HPA: 2+ replicas<br/>Replicas: ∞<br/>Memory: 512Mi"]
    end

    subgraph Tier4["Business Logic Tier"]
        subgraph Services["Core Services"]
            API["home-api-service:8081<br/>📊 Core API<br/>- User Management<br/>- Business Operations<br/>- Data CRUD<br/>- REST Endpoints<br/><br/>HPA: 2+ replicas<br/>Replicas: ∞<br/>Memory: 512Mi"]
            
            SEARCH["home-search-service:8081<br/>🔍 Search Engine<br/>- Full-text Search<br/>- Indexing<br/>- Query Processing<br/>- Data Discovery<br/><br/>HPA: 1+ replicas<br/>Replicas: ∞<br/>Memory: 512Mi<br/>Node: microk8s-worker"]
            
            IOT_TEL["iot-telemetry-service:8081<br/>📡 IoT Data Handler<br/>- Sensor Data Ingestion<br/>- Data Validation<br/>- Transformation<br/>- Aggregation<br/><br/>HPA: 1+ replicas<br/>Replicas: ∞<br/>Memory: 512Mi<br/>Node: khbr"]
        end
    end

    subgraph Tier5["Data Processing Tier"]
        ETL["home-etl-service:8081<br/>⚙️ ETL Pipeline<br/>- Batch Processing<br/>- Scheduled Jobs<br/>- Data Integration<br/>- Report Generation<br/><br/>StatefulSet: 1 replica<br/>Memory: 512Mi<br/>Node: jgte"]
        
        MQTT["mosquitto-service:1883<br/>🔀 Message Broker<br/>- MQTT Protocol<br/>- Message Queue<br/>- Pub/Sub Topics<br/>- TLS/SSL Support<br/><br/>NodePort: 31883<br/>Deployment: 1 replica<br/>Node: khbr"]
    end

    subgraph Tier6["Data Tier"]
        MYSQL["MySQL Database:3306<br/>💾 Data Storage<br/>- Persistent Storage<br/>- 3Gi Volume<br/>- MySQL 8.0-oracle<br/>- Multi-schema DB<br/><br/>StatefulSet: 1 replica<br/>NodePort: 32306<br/>Node: jgte<br/>Path: /home/alok/data/mysql"]
    end

    subgraph Tier7["Configuration & Security"]
        CFG["ConfigMaps<br/>5 Maps:<br/>- home-api-config<br/>- home-auth-config<br/>- home-email-config<br/>- iot-telemetry-config<br/>- mosquitto-config"]
        
        SEC["Secrets<br/>3 Sets:<br/>- mysql-secrets<br/>- iot-telemetry-secret<br/>- mosquitto-secret/ca/acl"]
    end

    %% Tier connections
    DASHBOARD -->|HTTP:80| INGRESS
    
    INGRESS -->|/home/auth| AUTH
    INGRESS -->|/home/api| API
    INGRESS -->|/home/search| SEARCH
    INGRESS -->|/home/etl| ETL
    INGRESS -->|/iot/telemetry| IOT_TEL
    INGRESS -->|MQTT| MQTT
    
    %% Service interactions
    AUTH -->|Validates Tokens| API
    AUTH -->|Validates Tokens| SEARCH
    AUTH -->|Validates Tokens| IOT_TEL
    AUTH -->|Validates Tokens| ETL
    
    API -->|Search Queries| SEARCH
    API -->|Subscribe Events| MQTT
    API -->|Triggers| ETL
    
    IOT_TEL -->|Send to Queue| MQTT
    IOT_TEL -->|Enrich & Process| API
    
    ETL -->|API Calls| API
    ETL -->|Data Sync| SEARCH
    
    %% Database connections
    AUTH -->|Read/Write| MYSQL
    API -->|Read/Write| MYSQL
    SEARCH -->|Read| MYSQL
    IOT_TEL -->|Write Metrics| MYSQL
    ETL -->|Transform Data| MYSQL
    
    %% Configuration
    AUTH -.->|Mount| CFG
    API -.->|Mount| CFG
    SEARCH -.->|Mount| CFG
    IOT_TEL -.->|Mount| CFG
    ETL -.->|Mount| CFG
    MQTT -.->|Mount| CFG
    
    AUTH -.->|Environment| SEC
    API -.->|Environment| SEC
    SEARCH -.->|Environment| SEC
    IOT_TEL -.->|Environment| SEC
    ETL -.->|Environment| SEC
    MQTT -.->|Volume| SEC

    style Tier1 fill:#FFE082
    style Tier2 fill:#81C784
    style Tier3 fill:#64B5F6
    style Tier4 fill:#A1887F
    style Tier5 fill:#F48FB1
    style Tier6 fill:#E57373
    style Tier7 fill:#D1D1D1
```

## Service Dependencies Map

```mermaid
graph LR
    AUTH["AUTH<br/>Token Provider"]
    API["API<br/>Core Logic"]
    SEARCH["SEARCH<br/>Indexer"]
    IOT["IOT<br/>Collector"]
    ETL["ETL<br/>Processor"]
    MQTT["MQTT<br/>Queue"]
    DB["MYSQL<br/>Store"]

    AUTH -->|"validate()"| API
    AUTH -->|"validate()"| SEARCH
    AUTH -->|"validate()"| IOT
    AUTH -->|"validate()"| ETL
    
    API -->|"search()"| SEARCH
    API -->|"publish()"| MQTT
    API -->|"trigger()"| ETL
    
    IOT -->|"publish()"| MQTT
    IOT -->|"enrich()"| API
    
    ETL -->|"query()"| API
    ETL -->|"sync()"| SEARCH
    
    API -->|"CRUD"| DB
    AUTH -->|"CRUD"| DB
    SEARCH -->|"SELECT"| DB
    IOT -->|"INSERT"| DB
    ETL -->|"SELECT/UPDATE"| DB
    MQTT -->|"log events"| DB

    style AUTH fill:#64B5F6
    style API fill:#A1887F
    style SEARCH fill:#A1887F
    style IOT fill:#F48FB1
    style ETL fill:#F48FB1
    style MQTT fill:#FFD54F
    style DB fill:#EF5350
```

## Namespace Isolation

```mermaid
graph TB
    subgraph K8S["Kubernetes Cluster"]
        subgraph NS_HS["home-stack<br/>(Application Services)"]
            SVC1["Services:<br/>- home-api-service<br/>- home-auth-service<br/>- home-search-service<br/>- home-etl-service<br/><br/>Network Label:<br/>network/db-access: true"]
        end

        subgraph NS_IOT["home-stack-iot<br/>(IoT Services)"]
            SVC2["Services:<br/>- iot-telemetry-service<br/>- mosquitto-service<br/><br/>Network Label:<br/>network/db-access: true"]
        end

        subgraph NS_DB["home-stack-db<br/>(Database)"]
            SVC3["Services:<br/>- mysql-service<br/><br/>Data:<br/>- PersistentVolume<br/>- PersistentVolumeClaim"]
        end

        subgraph NS_DMZ["home-stack-dmz<br/>(DMZ/Ingress)"]
            SVC4["Ingress Resources:<br/>- ingress-home-jgte<br/>- ingress-home-cloudflare<br/>- ingress-home-vhost<br/><br/>RBAC:<br/>- cluster-admin-user<br/>- dashboard-admin-user"]
        end

        NETPOL["NetworkPolicy<br/>Label: network/db-access<br/>Allows pod-to-DB access"]
    end

    SVC1 -->|"Cross-namespace"| NS_DB
    SVC2 -->|"Cross-namespace"| NS_DB
    NS_DMZ -->|"Routes traffic"| SVC1
    NS_DMZ -->|"Routes traffic"| SVC2
    NETPOL -.->|"Enforced on"| SVC1
    NETPOL -.->|"Enforced on"| SVC2

    style NS_HS fill:#E8F5E9
    style NS_IOT fill:#E3F2FD
    style NS_DB fill:#FFEBEE
    style NS_DMZ fill:#F3E5F5
    style NETPOL fill:#FFF3E0
```

## Pod Lifecycle & Health Checks

```
Startup Flow:
┌─────────────────────────────────────────────────┐
│ 1. Pod Created & Container Started               │
│    └─ Image pulled (Always)                      │
│    └─ Container entrypoint executed              │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 2. initialDelaySeconds (varies by service)       │
│    ├─ API:    5-10s (quick startup)              │
│    ├─ AUTH:   60-90s (slower startup)            │
│    ├─ SEARCH: 60s                                │
│    └─ IOT:    60s                                │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 3. Liveness Probe Checks (Ongoing)               │
│    └─ /actuator/health/liveness endpoint         │
│    └─ Period: 10-60s (per service)               │
│    └─ Timeout: 5s                                │
│    └─ Failure Threshold: 5-10 attempts           │
│    └─ Action: Restart container if fails         │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 4. Readiness Probe Checks (Ongoing)              │
│    └─ /actuator/health/readiness endpoint        │
│    └─ Period: 30-60s (per service)               │
│    └─ Timeout: 5s                                │
│    └─ Failure Threshold: 3-10 attempts           │
│    └─ Action: Remove from Service LB if fails    │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 5. Pod Ready (Receives Traffic)                  │
│    └─ Service routes requests to this pod        │
│    └─ Continues health checks                    │
│    └─ Can be scaled by HPA                       │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 6. Termination (Graceful)                        │
│    ├─ SIGTERM signal sent                        │
│    ├─ gracePeriod: 30s (default)                 │
│    ├─ Service stops routing traffic              │
│    └─ Application cleanups connections           │
└─────────────────────────────────────────────────┘
```

## Storage Architecture

```mermaid
graph LR
    APP["Application Pod<br/>mysql Container"]
    
    PVC["PersistentVolumeClaim<br/>mysql-pv-claim<br/>3Gi<br/>accessMode:<br/>ReadWriteOnce<br/>storageClass: manual"]
    
    PV["PersistentVolume<br/>mysql-pv-volume<br/>3Gi<br/>accessMode:<br/>ReadWriteOnce<br/>storageClass: manual"]
    
    FS["Host Filesystem<br/>/home/alok/data/mysql"]
    
    APP -->|"Mounts"| PVC
    PVC -->|"Binds 1:1"| PV
    PV -->|"hostPath"| FS

    style APP fill:#C8E6C9
    style PVC fill:#BBDEFB
    style PV fill:#FFCCBC
    style FS fill:#EF5350
```

## Configuration Injection Methods

### 1. Environment Variables (Secrets)
```yaml
env:
  - name: SPRING_DATASOURCE_USERNAME
    valueFrom:
      secretKeyRef:
        name: mysql-secrets
        key: stmt-user-name
```

### 2. Volume Mounts (TLS Certificates)
```yaml
volumeMounts:
  - name: iot-telemetry-secret
    mountPath: "/etc/jks"
    readOnly: true
```

### 3. ConfigMapRef (Bulk Configuration)
```yaml
envFrom:
  - configMapRef:
      name: iot-telemetry-config
```

## Key Characteristics

| Aspect | Details |
|--------|---------|
| **Cluster Type** | Kubernetes (MicroK8s or similar) |
| **Deployment Pattern** | Microservices with service mesh-less architecture |
| **Scaling Strategy** | HPA for stateless services, Fixed replicas for stateful |
| **Storage** | StatefulSets with hostPath PV (single node) |
| **Security** | Network policies, Secrets, RBAC |
| **HA Approach** | Multi-node cluster with affinity rules |
| **Observability** | Spring Boot Actuator endpoints for health |
| **Update Strategy** | Rolling updates for zero-downtime deployments |

## Critical Dependencies

1. **External Services**
   - Google OAuth for authentication
   - CloudFlare DNS for domain routing

2. **Internal Dependencies**
   - All services depend on MySQL
   - Auth service is authentication backbone
   - MQTT broker for IoT communication

3. **Infrastructure**
   - Persistent storage (local path provisioner)
   - Ingress controller (NGINX)
   - Network policies
   - RBAC configuration
