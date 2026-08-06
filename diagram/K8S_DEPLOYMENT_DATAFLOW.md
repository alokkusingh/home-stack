# Kubernetes Deployment Details & Data Flow

## Deployment Topology

```mermaid
graph TB
    subgraph K8S_Cluster["Kubernetes Cluster"]
        subgraph Nodes["Cluster Nodes"]
            subgraph JGTE["Node: jgte<br/>(Master/Primary)"]
                MYSQL_N["MySQL StatefulSet<br/>- Persistent Storage<br/>- Primary DB"]
                ETL_N["home-etl-service<br/>- StatefulSet<br/>- Batch Processing"]
            end

            subgraph KHBR["Node: khbr<br/>(Worker)"]
                SEARCH_N["home-search-service<br/>- Multiple replicas<br/>- Distributed"]
                IOT_N["iot-telemetry-service<br/>- Data Collection<br/>- Analytics"]
                MQTT_N["mosquitto-service<br/>- MQTT Broker<br/>- Message Queue"]
            end

            subgraph JGTE2["Node: jgte<br/>(Additional)"]
                API_N["home-api-service<br/>- HPA Replicas<br/>- Core Business Logic"]
            end

            subgraph Flexible["Nodes: Flexible<br/>(Auto-Assigned)"]
                AUTH_N["home-auth-service<br/>- HPA Replicas<br/>- Authentication"]
            end
        end

        subgraph Namespaces["Kubernetes Namespaces"]
            NS1["home-stack<br/>(API Services)"]
            NS2["home-stack-iot<br/>(IoT Services)"]
            NS3["home-stack-db<br/>(Database)"]
            NS4["home-stack-dmz<br/>(Ingress/DMZ)"]
        end

        subgraph Storage["Storage Resources"]
            PVC_NS["PVC: mysql-pv-claim<br/>3Gi<br/>StorageClass: manual"]
            PV_NS["PV: mysql-pv-volume<br/>hostPath: /home/alok/data/mysql<br/>accessMode: ReadWriteOnce"]
        end
    end

    MYSQL_N -.->|Claims| PVC_NS
    PVC_NS -.->|Binds| PV_NS
```

## Data Flow Architecture

```mermaid
graph LR
    subgraph Clients["Data Sources"]
        WEB_USER["Web Users"]
        MOBILE["Mobile Clients"]
        IOT_SENSORS["IoT Devices<br/>MQTT:1883"]
    end

    subgraph Ingestion["Data Ingestion"]
        ING_HTTP["HTTP/HTTPS<br/>Ingress Controller"]
        ING_MQTT["MQTT Broker<br/>mosquitto"]
    end

    subgraph Processing["Processing & Business Logic"]
        AUTH["Authentication<br/>home-auth-service<br/>- OAuth/JWT<br/>- Session Mgmt"]
        
        API["Core API<br/>home-api-service<br/>- User Management<br/>- Business Logic<br/>- API Endpoints"]
        
        SEARCH["Search Service<br/>home-search-service<br/>- Full-text Search<br/>- Indexing<br/>- Query Processing"]
        
        IOT_TEL["IoT Telemetry<br/>iot-telemetry-service<br/>- Data Collection<br/>- Validation<br/>- Transformation"]
        
        ETL["ETL Pipeline<br/>home-etl-service<br/>- Data Integration<br/>- Batch Processing<br/>- Schedule Jobs"]
    end

    subgraph Database["Data Storage"]
        MYSQL["MySQL Database<br/>- User Data<br/>- Business Records<br/>- IoT Metrics<br/>- Search Index"]
    end

    subgraph Output["Data Output"]
        DASH["Dashboard"]
        ANALYTICS["Analytics"]
        REPORTS["Reports"]
    end

    WEB_USER -->|Login/API Calls| ING_HTTP
    MOBILE -->|Login/API Calls| ING_HTTP
    IOT_SENSORS -->|Sensor Data| ING_MQTT
    
    ING_HTTP -->|Route| AUTH
    AUTH -->|Validate| API
    AUTH -->|Token Check| SEARCH
    
    ING_HTTP -->|Route| API
    ING_HTTP -->|Route| SEARCH
    ING_MQTT -->|Messages| IOT_TEL
    
    API -->|Read/Write| MYSQL
    SEARCH -->|Query| MYSQL
    IOT_TEL -->|Store Metrics| MYSQL
    ETL -->|Read/Transform| MYSQL
    ETL -->|Write Results| MYSQL
    
    MYSQL -->|Data| DASH
    MYSQL -->|Metrics| ANALYTICS
    ETL -->|Generate| REPORTS

    style Clients fill:#E3F2FD
    style Ingestion fill:#F3E5F5
    style Processing fill:#E8F5E9
    style Database fill:#FFEBEE
    style Output fill:#FFF3E0
```

## Service Communication Map

```mermaid
graph TB
    subgraph External["External"]
        USER["Users"]
        DEVICES["IoT Devices"]
    end

    subgraph EntryPoint["Entry Points"]
        INGRESS["Ingress Controller<br/>3 Rules:<br/>- jgte<br/>- hdash.alok.world<br/>- alok-home.com"]
    end

    subgraph ServiceLayer["Service Layer"]
        API_SVC["home-api-service:8081"]
        AUTH_SVC["home-auth-service:8081"]
        SEARCH_SVC["home-search-service:8081"]
        ETL_SVC["home-etl-service:8081"]
        IOT_SVC["iot-telemetry-service:8081"]
        MQTT_SVC["mosquitto-service:1883"]
    end

    subgraph Dependencies["Dependencies"]
        DB_SVC["mysql:3306"]
        CFG["ConfigMaps<br/>5 maps"]
        SEC["Secrets<br/>3 sets"]
    end

    USER -->|HTTP/HTTPS| INGRESS
    DEVICES -->|MQTT| MQTT_SVC
    
    INGRESS -->|/home/api| API_SVC
    INGRESS -->|/home/auth| AUTH_SVC
    INGRESS -->|/home/search| SEARCH_SVC
    INGRESS -->|/home/etl| ETL_SVC
    
    API_SVC -->|authenticate| AUTH_SVC
    API_SVC -->|query| SEARCH_SVC
    SEARCH_SVC -->|authenticate| AUTH_SVC
    ETL_SVC -->|api calls| API_SVC
    IOT_SVC -->|mqtt broker| MQTT_SVC
    
    API_SVC -->|DB Queries| DB_SVC
    AUTH_SVC -->|DB Queries| DB_SVC
    SEARCH_SVC -->|DB Queries| DB_SVC
    ETL_SVC -->|DB Queries| DB_SVC
    IOT_SVC -->|DB Queries| DB_SVC
    
    API_SVC -.->|Config| CFG
    AUTH_SVC -.->|Config| CFG
    SEARCH_SVC -.->|Config| CFG
    ETL_SVC -.->|Config| CFG
    IOT_SVC -.->|Config| CFG
    
    API_SVC -.->|Secrets| SEC
    AUTH_SVC -.->|Secrets| SEC
    SEARCH_SVC -.->|Secrets| SEC
    ETL_SVC -.->|Secrets| SEC
    IOT_SVC -.->|Secrets| SEC
    MQTT_SVC -.->|Secrets| SEC

    style External fill:#BBDEFB
    style EntryPoint fill:#E1BEE7
    style ServiceLayer fill:#C8E6C9
    style Dependencies fill:#FFF9C4
```

## Port & Network Mapping

### Internal Communication
```
Within Cluster:
├── home-api-service.home-stack:8081
├── home-auth-service.home-stack:8081
├── home-search-service.home-stack:8081
├── home-etl-service.home-stack:8081
├── iot-telemetry-service.home-stack-iot:8081
├── mosquitto-service.home-stack-iot:1883
└── mysql.home-stack-db:3306
```

### External Access
```
NodePort Services:
├── MQTT Broker:       node:31883  -> mosquitto:1883
├── MySQL Database:    node:32306  -> mysql:3306
└── HTTP/HTTPS:        LoadBalancer -> Ingress:80/443
```

### DNS Resolution
```
Internal (ClusterDNS):
├── home-api-service.home-stack.svc.cluster.local:8081
├── home-auth-service.home-stack.svc.cluster.local:8081
├── home-search-service.home-stack.svc.cluster.local:8081
├── home-etl-service.home-stack.svc.cluster.local:8081
├── iot-telemetry-service.home-stack-iot.svc.cluster.local:8081
├── mosquitto-service.home-stack-iot.svc.cluster.local:1883
└── mysql.home-stack-db.svc.cluster.local:3306

External (CloudFlare):
├── hdash.alok.world:443 -> Ingress LoadBalancer
└── alok-home.com:443    -> Ingress LoadBalancer
```

## Scaling & Resource Allocation

### HPA Configuration
```
home-api-service:
├── Min Replicas:    2
├── Max Replicas:    N (defined in home-hpa.yaml)
├── CPU Target:      ~80%
└── Memory Target:   ~80%

home-auth-service:
├── Min Replicas:    2
├── Max Replicas:    N
├── CPU Target:      ~80%
└── Memory Target:   ~80%

home-search-service:
├── Min Replicas:    1
├── Max Replicas:    N
├── CPU Target:      ~80%
└── Memory Target:   ~80%
```

### Resource Requests & Limits
```
Standard Spring Boot Service:
├── CPU:
│   ├── Request:    100-200m
│   └── Limit:      Not enforced (burstable)
├── Memory:
│   ├── Request:    256Mi
│   └── Limit:      512Mi
└── Image:          Always pulled (latest)

Stateful Services:
├── home-etl:       1 fixed replica
└── mysql:          1 fixed replica
```

## Network Security

### Network Policies
- **Label-based access**: `network/db-access: "true"`
- **Services with DB access**: All application services

### Service Types
```
ClusterIP (Internal):
├── home-api-service
├── home-auth-service
├── home-search-service
├── home-etl-service
├── iot-telemetry-service
└── mysql (primary)

NodePort (External):
├── mosquitto-service:1883  (31883)
└── mysql:3306              (32306)
```

## Deployment Strategy

### Rolling Updates
```
Update Strategy: RollingUpdate
├── All services use rolling updates
├── Pods replaced gradually
├── Old pods terminate after new ready
└── Zero-downtime deployments
```

### Health Verification
```
Service Startup Sequence:
1. Pod starts container
2. Wait initialDelaySeconds
3. First liveness probe
4. First readiness probe
5. Service traffic enabled (readiness pass)
6. Periodic health checks continue
7. Restart on liveness failure
8. Remove from service on readiness failure
```
