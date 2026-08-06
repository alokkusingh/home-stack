# Kubernetes Component Architecture - Home Stack

## Detailed Component Diagram

```mermaid
graph LR
    subgraph Client["Client Layer"]
        WEB["Web Browser<br/>Mobile Client"]
        IOT_DEV["IoT Devices<br/>MQTT Clients"]
    end

    subgraph External["External Services"]
        CF_DNS["CloudFlare DNS<br/>hdash.alok.world"]
        GOOGLE_OAUTH["Google OAuth<br/>Provider"]
    end

    subgraph Ingress_Layer["Kubernetes Ingress Layer"]
        ING["NGINX Ingress Controller"]
        LB["Load Balancer"]
    end

    subgraph API_Layer["API Gateway & Services"]
        subgraph home_stack_ns["home-stack Namespace"]
            API_SVC["home-api-service<br/>ClusterIP:8081"]
            AUTH_SVC["home-auth-service<br/>ClusterIP:8081"]
            SEARCH_SVC["home-search-service<br/>ClusterIP:8081"]
            ETL_SVC["home-etl-service<br/>ClusterIP:8081"]
        end

        subgraph home_stack_iot_ns["home-stack-iot Namespace"]
            IOT_SVC["iot-telemetry-service<br/>ClusterIP:8081"]
            MQTT_SVC["mosquitto-service<br/>NodePort:31883"]
        end
    end

    subgraph Business_Logic["Business Logic Pods"]
        subgraph api_pods["home-api Pods"]
            API_P1["Pod 1: home-api<br/>Port 8081<br/>512Mi RAM"]
            API_P2["Pod 2: home-api<br/>Port 8081<br/>512Mi RAM"]
            API_P3["Pod 3: home-api<br/>Port 8081<br/>512Mi RAM"]
        end

        subgraph auth_pods["home-auth Pods"]
            AUTH_P1["Pod 1: home-auth<br/>Port 8081<br/>512Mi RAM"]
            AUTH_P2["Pod 2: home-auth<br/>Port 8081<br/>512Mi RAM"]
        end

        subgraph search_pods["home-search Pods"]
            SEARCH_P1["Pod 1: home-search<br/>Port 8081<br/>512Mi RAM"]
        end

        subgraph etl_pods["home-etl Pod"]
            ETL_P1["Pod 1: home-etl<br/>StatefulSet<br/>Port 8081<br/>512Mi RAM"]
        end

        subgraph iot_pods["IoT Pods"]
            IOT_P1["Pod 1: iot-telemetry<br/>Port 8081<br/>512Mi RAM"]
            MQTT_P1["Pod 1: mosquitto<br/>Port 1883<br/>v2.0.21"]
        end
    end

    subgraph Data_Persistence["Data Persistence Layer"]
        subgraph db_ns["home-stack-db Namespace"]
            MYSQL_SVC["mysql-service<br/>StatefulSet<br/>NodePort:32306"]
            MYSQL_POD["MySQL:8.0-oracle<br/>Port 3306"]
            PVC["PVC<br/>mysql-pv-claim<br/>3Gi"]
            PV["PV<br/>mysql-pv-volume<br/>/home/alok/data/mysql"]
        end
    end

    subgraph Config_Secrets["Configuration & Secrets"]
        CFG_API["ConfigMap<br/>home-api-config"]
        CFG_AUTH["ConfigMap<br/>home-auth-config"]
        CFG_EMAIL["ConfigMap<br/>home-email-config"]
        CFG_IOT["ConfigMap<br/>iot-telemetry-config"]
        CFG_MQTT["ConfigMap<br/>mosquitto-config"]
        
        SEC_DB["Secret<br/>mysql-secrets"]
        SEC_IOT["Secret<br/>iot-telemetry-secret"]
        SEC_MQTT["Secret<br/>mosquitto-secret/ca/acl"]
    end

    subgraph Monitoring["Monitoring & Observability"]
        HC["Health Checks<br/>Liveness & Readiness"]
        HPA["HPA Controller<br/>Auto-scaling"]
        METRICS["Metrics Server"]
    end

    %% Client connections
    WEB -->|HTTPS| CF_DNS
    IOT_DEV -->|MQTT:1883| MQTT_SVC
    WEB -->|OAuth| GOOGLE_OAUTH
    
    %% DNS to Ingress
    CF_DNS -->|Routes to| LB
    LB -->|Ingress Rules| ING
    
    %% Ingress to Services
    ING -->|Route /home/api| API_SVC
    ING -->|Route /home/auth| AUTH_SVC
    ING -->|Route /home/search| SEARCH_SVC
    ING -->|Route /home/etl| ETL_SVC
    
    %% Services to Pods
    API_SVC -->|Load Balance| API_P1
    API_SVC -->|Load Balance| API_P2
    API_SVC -->|Load Balance| API_P3
    
    AUTH_SVC -->|Load Balance| AUTH_P1
    AUTH_SVC -->|Load Balance| AUTH_P2
    
    SEARCH_SVC -->|Load Balance| SEARCH_P1
    
    ETL_SVC -->|Direct| ETL_P1
    
    IOT_SVC -->|Direct| IOT_P1
    MQTT_SVC -->|Direct| MQTT_P1
    
    %% Configuration injection
    API_P1 -.->|Mount| CFG_API
    API_P2 -.->|Mount| CFG_API
    API_P3 -.->|Mount| CFG_API
    
    AUTH_P1 -.->|Mount| CFG_AUTH
    AUTH_P2 -.->|Mount| CFG_AUTH
    
    SEARCH_P1 -.->|Mount| CFG_EMAIL
    ETL_P1 -.->|Mount| CFG_API
    
    IOT_P1 -.->|Mount| CFG_IOT
    MQTT_P1 -.->|Mount| CFG_MQTT
    
    %% Secrets injection
    API_P1 -.->|Environment| SEC_DB
    API_P2 -.->|Environment| SEC_DB
    API_P3 -.->|Environment| SEC_DB
    AUTH_P1 -.->|Environment| SEC_DB
    AUTH_P2 -.->|Environment| SEC_DB
    SEARCH_P1 -.->|Environment| SEC_DB
    ETL_P1 -.->|Environment| SEC_DB
    IOT_P1 -.->|Environment| SEC_DB
    IOT_P1 -.->|Volume Mount| SEC_IOT
    MQTT_P1 -.->|Volume Mount| SEC_MQTT
    
    %% Database connections
    API_P1 -->|TCP:3306| MYSQL_SVC
    API_P2 -->|TCP:3306| MYSQL_SVC
    API_P3 -->|TCP:3306| MYSQL_SVC
    AUTH_P1 -->|TCP:3306| MYSQL_SVC
    AUTH_P2 -->|TCP:3306| MYSQL_SVC
    SEARCH_P1 -->|TCP:3306| MYSQL_SVC
    ETL_P1 -->|TCP:3306| MYSQL_SVC
    IOT_P1 -->|TCP:3306| MYSQL_SVC
    
    MYSQL_SVC -->|Manages| MYSQL_POD
    MYSQL_POD -->|Mounts| PVC
    PVC -->|Claims| PV
    
    %% Health checks & Monitoring
    HC -.->|Monitors| API_P1
    HC -.->|Monitors| AUTH_P1
    HC -.->|Monitors| SEARCH_P1
    HC -.->|Monitors| ETL_P1
    HC -.->|Monitors| IOT_P1
    HC -.->|Monitors| MQTT_P1
    
    HPA -.->|Scales| API_SVC
    HPA -.->|Scales| AUTH_SVC
    HPA -.->|Scales| SEARCH_SVC
    
    METRICS -.->|Collects| API_P1
    METRICS -.->|Collects| MYSQL_POD

    style Client fill:#E1F5FF
    style External fill:#FFF3E0
    style Ingress_Layer fill:#F3E5F5
    style API_Layer fill:#E8F5E9
    style Business_Logic fill:#C8E6C9
    style Data_Persistence fill:#FFCCCC
    style Config_Secrets fill:#F0F0F0
    style Monitoring fill:#FCE4EC
    style home_stack_ns fill:#FFFFFF
    style home_stack_iot_ns fill:#FFFFFF
    style db_ns fill:#FFFFFF
```

## Component Details

### Ingress Layer
| Component | Type | Purpose |
|-----------|------|---------|
| NGINX Ingress Controller | Service Mesh | Routes external HTTP/HTTPS traffic |
| Load Balancer | Network | Distributes load across ingress controllers |
| Ingress Rules | K8s Resource | Define hostname-based routing rules |

### API Services
| Service | Port | Deployment Type | Replicas | Node Affinity |
|---------|------|-----------------|----------|--------------|
| home-api-service | 8081 | Deployment | HPA (2-N) | None (flexible) |
| home-auth-service | 8081 | Deployment | HPA (2-N) | None (flexible) |
| home-search-service | 8081 | Deployment | HPA (2-N) | microk8s-worker |
| home-etl-service | 8081 | StatefulSet | 1 | jgte |
| iot-telemetry-service | 8081 | Deployment | HPA (1-N) | khbr |
| mosquitto-service | 1883 | Deployment | 1 | khbr |

### Pod Specifications
```yaml
Resources per Pod:
  CPU:
    Request: 100-200m
    Limit: Unbounded

  Memory:
    Request: 256-512Mi
    Limit: 512Mi

Health Checks:
  Liveness Probe:
    - Endpoint: /actuator/health/liveness
    - InitialDelay: 5-60s
    - Period: 10-60s
    - Timeout: 5s
    - FailureThreshold: 5-10

  Readiness Probe:
    - Endpoint: /actuator/health/readiness
    - InitialDelay: 10-90s
    - Period: 30-60s
    - Timeout: 5s
    - FailureThreshold: 3-10

Image Pull Policy: Always
Volume Mounts: Timezone config (/usr/share/zoneinfo/Asia/Kolkata)
```

### Database Layer
- **Type**: MySQL 8.0-oracle
- **Deployment**: StatefulSet (persistent state)
- **Storage**: 3Gi PersistentVolume
- **Location**: /home/alok/data/mysql
- **Access**: ClusterIP (internal) + NodePort:32306 (external)
- **Node Pinning**: jgte (single node database)

### Configuration Management
- **ConfigMaps**: 5 maps for different services
- **Secrets**: 3 secret sets for:
  - Database credentials
  - IoT certificates
  - MQTT TLS/ACL
- **Injection Methods**:
  - Environment variables
  - Volume mounts
  - ConfigMapRef (bulk loading)

### Auto-scaling Strategy
- **HPA Configuration**:
  - Targets: home-api, home-auth, home-search
  - Metrics: CPU & Memory
  - Defined in: home-hpa.yaml

### Monitoring & Observability
- **Health Endpoints**: Spring Boot Actuator
- **Metrics Server**: Collects resource metrics
- **Horizontal Pod Autoscaler**: Responds to metrics
- **No explicit Service Mesh**: Direct pod-to-pod communication
