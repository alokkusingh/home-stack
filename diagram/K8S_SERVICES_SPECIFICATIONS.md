# Kubernetes Services - Detailed Specifications

## Services Summary Table

| Service | Type | Port | Node Affinity | Replicas | Memory | CPU | Status |
|---------|------|------|---------------|----------|--------|-----|--------|
| **home-api-service** | Deployment | 8081 | None | HPA | 512Mi | 200m req | ClusterIP |
| **home-auth-service** | Deployment | 8081 | None | HPA | 512Mi | 100m req | ClusterIP |
| **home-search-service** | Deployment | 8081 | microk8s-worker | HPA | 512Mi | 200m req | ClusterIP |
| **home-etl-service** | StatefulSet | 8081 | jgte | 1 | 512Mi | - | ClusterIP |
| **iot-telemetry-service** | Deployment | 8081 | khbr | HPA | 512Mi | 200m req | ClusterIP |
| **mosquitto-service** | Deployment | 1883 | khbr | 1 | - | - | NodePort(31883) |
| **mysql** | StatefulSet | 3306 | jgte | 1 | - | - | NodePort(32306) |

## Detailed Service Specifications

### 1. home-api-service

**Purpose**: Core API gateway and business logic

```yaml
Metadata:
  - Name: home-api-service
  - Namespace: home-stack
  - Service Type: ClusterIP

Port Configuration:
  - Service Port: 8081
  - Container Port: 8081
  - Protocol: TCP

Deployment:
  - Kind: Deployment
  - Strategy: RollingUpdate
  - Selector: app=home-api
  - Label: network/db-access=true

Pod Resources:
  - Memory Request: 256Mi
  - Memory Limit: 512Mi
  - CPU Request: 200m
  - CPU Limit: Unbounded

Health Checks:
  - Liveness Probe:
    * Endpoint: /home/api/actuator/health/liveness
    * InitialDelay: 5s
    * Period: 10s
    * Timeout: 5s
    * FailureThreshold: 5
  
  - Readiness Probe:
    * Endpoint: /home/api/actuator/health/readiness
    * InitialDelay: 10s
    * Period: 30s
    * Timeout: 5s
    * FailureThreshold: 3

Environment Variables (from ConfigMap):
  - SPRING_PROFILES_ACTIVE
  - SPRING_DATASOURCE_URL (db-url)
  - SPRING_DATASOURCE_USERNAME (secret: mysql-secrets)
  - SPRING_DATASOURCE_PASSWORD (secret: mysql-secrets)
  - LOGGING_LEVEL_* (multiple log levels)
  - SPRING_DATASOURCE_HIKARI_* (connection pool settings)
  - IOT_SECURE_* (truststore/keystore passwords)

Volume Mounts:
  - tz-config: /usr/share/zoneinfo/Asia/Kolkata

HPA Configuration:
  - Min Replicas: 2
  - Max Replicas: (See home-hpa.yaml)
  - Metrics: CPU & Memory
```

### 2. home-auth-service

**Purpose**: Authentication, Authorization, OAuth integration

```yaml
Metadata:
  - Name: home-auth-service
  - Namespace: home-stack
  - Service Type: ClusterIP

Port Configuration:
  - Service Port: 8081
  - Container Port: 8081
  - Protocol: TCP

Deployment:
  - Kind: Deployment
  - Strategy: RollingUpdate
  - Selector: app=home-auth
  - Label: network/db-access=true

Pod Resources:
  - Memory Request: 256Mi
  - Memory Limit: 512Mi
  - CPU Request: 100m
  - CPU Limit: Unbounded

Health Checks:
  - Liveness Probe:
    * Endpoint: /home/auth/actuator/health/liveness
    * InitialDelay: 60s
    * Period: 10s
    * Timeout: 5s
    * FailureThreshold: 10
  
  - Readiness Probe:
    * Endpoint: /home/auth/actuator/health/readiness
    * InitialDelay: 90s
    * Period: 30s
    * Timeout: 5s
    * FailureThreshold: 10

Environment Variables (from ConfigMap):
  - OAUTH_GOOGLE_CLIENT_ID
  - SPRING_DATASOURCE_URL (from home-api-cofig)
  - SPRING_DATASOURCE_USERNAME (secret: mysql-secrets)
  - SPRING_DATASOURCE_PASSWORD (secret: mysql-secrets)
  - APPLICATION_SECURITY_JWT_SECRET
  - APPLICATION_SECRET
  - SPRING_PROFILES_ACTIVE
  - POD_NAME (from fieldRef: metadata.name)
  - LOGGING_LEVEL_* (multiple log levels)

Volume Mounts:
  - tz-config: /usr/share/zoneinfo/Asia/Kolkata

HPA Configuration:
  - Min Replicas: 2
  - Max Replicas: (See home-hpa.yaml)
  - Metrics: CPU & Memory
```

### 3. home-search-service

**Purpose**: Full-text search, indexing, query processing

```yaml
Metadata:
  - Name: home-search-service
  - Namespace: home-stack
  - Service Type: ClusterIP

Port Configuration:
  - Service Port: 8081
  - Container Port: 8081
  - Protocol: TCP

Deployment:
  - Kind: Deployment
  - Strategy: RollingUpdate
  - Selector: app=home-search-p
  - Label: network/db-access=true

Pod Resources:
  - Memory Request: 256Mi
  - Memory Limit: 512Mi
  - CPU Request: 200m
  - CPU Limit: Unbounded

Node Affinity:
  - Preferred: node.kubernetes.io/microk8s-worker=microk8s-worker
  - Weight: 1

Health Checks:
  - Liveness Probe:
    * Endpoint: /home/search/actuator/health/liveness
    * InitialDelay: 60s
    * Period: 60s
    * Timeout: 5s
    * FailureThreshold: 5
  
  - Readiness Probe:
    * Endpoint: /home/search/actuator/health/readiness
    * InitialDelay: 60s
    * Period: 60s
    * Timeout: 5s
    * FailureThreshold: 3

Environment Variables (from ConfigMap: home-email-cofig):
  - SPRING_PROFILES_ACTIVE
  - SPRING_DATASOURCE_URL (db-url)
  - SPRING_DATASOURCE_USERNAME (secret: mysql-secrets)
  - SPRING_DATASOURCE_PASSWORD (secret: mysql-secrets)
  - LOGGING_LEVEL_* (multiple log levels)

Volume Mounts:
  - tz-config: /usr/share/zoneinfo/Asia/Kolkata

HPA Configuration:
  - Min Replicas: 1
  - Max Replicas: (See home-hpa.yaml)
  - Metrics: CPU & Memory
```

### 4. home-etl-service

**Purpose**: ETL pipeline, batch processing, scheduled jobs

```yaml
Metadata:
  - Name: home-etl-service
  - Namespace: home-stack
  - Service Type: ClusterIP

Port Configuration:
  - Service Port: 8081
  - Container Port: 8081
  - Protocol: TCP

Deployment:
  - Kind: StatefulSet
  - ServiceName: home-etl-service
  - Strategy: N/A (StatefulSet)
  - Selector: app=home-etl
  - Label: network/db-access=true

Pod Resources:
  - Memory Request: Not specified
  - Memory Limit: Not specified
  - CPU Request: Not specified
  - CPU Limit: Not specified

Node Selector:
  - kubernetes.io/hostname: jgte (Fixed to jgte node)

Health Checks:
  - Liveness Probe:
    * Endpoint: /home/etl/actuator/health/liveness
    * InitialDelay: 60s
    * Period: 15s
    * Timeout: 5s
    * FailureThreshold: 10
  
  - Readiness Probe:
    * Endpoint: /home/etl/actuator/health/readiness
    * InitialDelay: 60s
    * Period: 30s (inferred)
    * Timeout: 5s
    * FailureThreshold: 3 (inferred)

Environment Variables (from ConfigMap):
  - SPRING_PROFILES_ACTIVE
  - SPRING_DATASOURCE_URL
  - SPRING_DATASOURCE_USERNAME (secret: mysql-secrets)
  - SPRING_DATASOURCE_PASSWORD (secret: mysql-secrets)
  - LOGGING_LEVEL_* (multiple log levels)

Volume Mounts:
  - tz-config: /usr/share/zoneinfo/Asia/Kolkata

Scaling:
  - Fixed Replicas: 1 (StatefulSet - no autoscaling)
  - Reason: Stateful workload with scheduled jobs
```

### 5. iot-telemetry-service

**Purpose**: IoT data collection, telemetry processing, metrics aggregation

```yaml
Metadata:
  - Name: iot-telemetry-service
  - Namespace: home-stack-iot
  - Service Type: ClusterIP

Port Configuration:
  - Service Port: 8081
  - Container Port: 8081
  - Protocol: TCP

Deployment:
  - Kind: Deployment
  - Strategy: RollingUpdate
  - Selector: app=iot-telemetry
  - Label: network/db-access=true

Pod Resources:
  - Memory Request: 256Mi
  - Memory Limit: 512Mi
  - CPU Request: 200m
  - CPU Limit: Unbounded

Node Selector:
  - kubernetes.io/hostname: khbr (Fixed to khbr node)

Volume Mounts:
  - iot-telemetry-secret: /etc/jks (readonly)
  - mosquitto-ca-secret: /etc/ca (readonly)

Environment Variables (from ConfigMap: iot-telemetry-config):
  - All variables from configMap (envFrom)
  - KSPASSWORD (specific override)
  - SPRING_DATASOURCE_USERNAME (secret: mysql-secrets)
  - SPRING_DATASOURCE_PASSWORD (secret: mysql-secrets)

Health Checks:
  - Liveness Probe:
    * Endpoint: /home/telemetry/actuator/health/liveness
    * InitialDelay: 60s
    * Period: 10s
    * Timeout: 5s
    * FailureThreshold: 5
  
  - Readiness Probe:
    * Endpoint: /home/telemetry/actuator/health/readiness
    * InitialDelay: 60s
    * Period: 30s
    * Timeout: 5s
    * FailureThreshold: 3

HPA Configuration:
  - Min Replicas: 1
  - Max Replicas: (See home-hpa.yaml)
  - Metrics: CPU & Memory
```

### 6. mosquitto-service

**Purpose**: MQTT message broker for IoT communication

```yaml
Metadata:
  - Name: mosquitto-service
  - Namespace: home-stack-iot
  - Service Type: NodePort

Port Configuration:
  - Service Port: 1883
  - Container Port: 1883
  - NodePort: 31883 (external access)
  - Protocol: TCP
  - Name: mqtt

Deployment:
  - Kind: Deployment
  - Selector: app=mosquitto
  - Replicas: 1 (fixed)

Image:
  - eclipse-mosquitto:2.0.21

Node Selector:
  - kubernetes.io/hostname: khbr (Fixed to khbr node)

Volume Mounts:
  - mosquitto-config: /mosquitto/config (readonly)
  - mosquitto-secret: /etc/tls (readonly - TLS certs)
  - mosquitto-ca-secret: /etc/ca (readonly - CA certs)
  - mosquitto-acl-secret: /etc/acl (readonly - ACL config)

Resource Management:
  - Resources: Not specified (default unlimited)

Health Checks:
  - No explicit probes (not a Spring Boot app)
```

### 7. mysql-service

**Purpose**: Primary data store for all services

```yaml
Metadata:
  - Name: mysql
  - Namespace: home-stack-db
  - Service Type: NodePort

Port Configuration:
  - Service Port: 3306 (internal)
  - Container Port: 3306
  - NodePort: 32306 (external access)
  - Protocol: TCP

Deployment:
  - Kind: StatefulSet
  - ServiceName: mysql
  - Selector: app=mysql
  - Replicas: 1 (fixed - single database)

Image:
  - mysql:8.0-oracle

Node Selector:
  - kubernetes.io/hostname: jgte (Fixed to jgte node)

Environment Variables:
  - MYSQL_ROOT_PASSWORD (secret: mysql-secrets)
  - MYSQL_ROOT_HOST: '%' (allow remote access)

Volume Mounts:
  - mysql-persistent-storage: /var/lib/mysql
  - tz-config: /usr/share/zoneinfo/Asia/Kolkata

Storage:
  - PersistentVolumeClaim: mysql-pv-claim (3Gi)
  - PersistentVolume: mysql-pv-volume
  - StorageClass: manual
  - AccessMode: ReadWriteOnce
  - HostPath: /home/alok/data/mysql

Resource Management:
  - Resources: Not specified
  - Run as: root (default for MySQL)

Health Checks:
  - No explicit probes
  - Relies on service connectivity
```

## Service Communication Matrix

```
Source              → Destination           Method          Auth Required
────────────────────────────────────────────────────────────────────────
External Clients    → Ingress Controller     HTTP/HTTPS      N/A
Ingress             → home-api-service      Internal DNS    No
Ingress             → home-auth-service     Internal DNS    No
Ingress             → home-search-service   Internal DNS    No
Ingress             → home-etl-service      Internal DNS    No

home-api-service    → home-auth-service     Internal DNS    Yes (JWT)
home-api-service    → home-search-service   Internal DNS    Yes (JWT)
home-api-service    → mysql                 Internal DNS    DB User/Pass
home-api-service    → mosquitto-service     Internal DNS    No

home-auth-service   → mysql                 Internal DNS    DB User/Pass
home-auth-service   → Google OAuth          External DNS    OAuth Token

home-search-service → mysql                 Internal DNS    DB User/Pass
home-search-service → home-auth-service     Internal DNS    Yes (JWT)

home-etl-service    → mysql                 Internal DNS    DB User/Pass
home-etl-service    → home-api-service      Internal DNS    No (internal)

iot-telemetry-service → mysql               Internal DNS    DB User/Pass
iot-telemetry-service → mosquitto-service   Internal DNS    No

mosquitto-service   ← IoT Devices           MQTT:1883       Yes (ACL)
```

## Critical Configurations

### Network Policies
- Label-based: `network/db-access: true`
- Restricts database access to authorized services

### Secrets Used
```
mysql-secrets:
  - stmt-user-name
  - stmt-password
  - root-password

iot-telemetry-secret:
  - JKS keystore for IoT communication

mosquitto-secret/ca/acl:
  - TLS certificates
  - CA certificates
  - ACL configuration
```

### ConfigMaps Used
- `home-api-cofig` - API configuration
- `home-auth-cofig` - Auth configuration
- `home-email-cofig` - Email/Search configuration
- `iot-telemetry-config` - IoT configuration
- `mosquitto-config` - MQTT broker configuration

## Resource Allocation Summary

```
Total Reserved Resources (at minimum replicas):
├── CPU Request: ~1.2 cores
│   ├── home-api: 200m × 2 = 400m
│   ├── home-auth: 100m × 2 = 200m
│   ├── home-search: 200m × 1 = 200m
│   └── iot-telemetry: 200m × 1 = 200m
│
└── Memory Request: ~3.5Gi (at 2 replicas each)
    ├── home-api: 256m × 2 = 512m
    ├── home-auth: 256m × 2 = 512m
    ├── home-search: 256m × 1 = 256m
    ├── iot-telemetry: 256m × 1 = 256m
    └── Total: ~1.5Gi minimum

Limits:
├── Memory Limit (all services): 512Mi × 6 services = 3Gi
└── No CPU limits enforced (burstable QoS)
```

## Deployment Best Practices Implemented

✅ Health checks (Liveness & Readiness)
✅ Resource requests & limits
✅ Rolling update strategy
✅ Node affinity for critical services
✅ StatefulSets for stateful workloads
✅ Persistent storage for databases
✅ Configuration externalization (ConfigMaps & Secrets)
✅ Network policies for security
✅ Multi-namespace architecture for isolation
✅ Service selectors for pod routing
✅ NodePort for external access
✅ Image pull policy (Always - for latest updates)
