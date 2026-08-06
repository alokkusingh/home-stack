# Kubernetes Architecture Documentation - Home Stack

## 📋 Documentation Index

This comprehensive documentation covers the complete Kubernetes architecture for the Home Stack project.

### Documents Included:

1. **K8S_ARCHITECTURE_OVERVIEW.md** 
   - Complete system architecture diagram
   - Namespace overview
   - Service topology
   - Configuration and security
   - Network isolation

2. **K8S_COMPONENT_DIAGRAM.md**
   - Detailed component architecture
   - Pod specifications
   - Resource allocations
   - Auto-scaling strategy
   - Monitoring and observability

3. **K8S_DEPLOYMENT_DATAFLOW.md**
   - Deployment topology across nodes
   - Data flow architecture
   - Service communication map
   - Port and network mapping
   - DNS resolution paths
   - Scaling and resource allocation
   - Network security details
   - Deployment strategies

4. **K8S_APPLICATION_LAYERS.md**
   - 7-tier application stack diagram
   - Service dependencies
   - Namespace isolation
   - Pod lifecycle and health checks
   - Storage architecture
   - Configuration injection methods

5. **K8S_SERVICES_SPECIFICATIONS.md**
   - Detailed service specifications
   - Per-service configuration
   - Resource management
   - Health check settings
   - Environment variables
   - Communication matrix
   - Best practices implemented

---

## 🏗️ Architecture Summary

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Kubernetes Cluster                          │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              Ingress Layer (DMZ)                          │ │
│  │  - NGINX Ingress Controller                              │ │
│  │  - 3 Ingress Rules                                       │ │
│  │  - Path-based routing                                    │ │
│  └───────────────────────────────────────────────────────────┘ │
│         ↓         ↓         ↓           ↓          ↓            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           Application Services Layer                     │  │
│  │  ┌────────────────────────────────────────────────────┐ │  │
│  │  │ home-stack Namespace (Main Services)              │ │  │
│  │  │ - home-api-service (HPA)                          │ │  │
│  │  │ - home-auth-service (HPA)                         │ │  │
│  │  │ - home-search-service (HPA)                       │ │  │
│  │  │ - home-etl-service (StatefulSet: 1)              │ │  │
│  │  └────────────────────────────────────────────────────┘ │  │
│  │  ┌────────────────────────────────────────────────────┐ │  │
│  │  │ home-stack-iot Namespace (IoT Services)           │ │  │
│  │  │ - iot-telemetry-service (HPA)                     │ │  │
│  │  │ - mosquitto-service (Deployment: 1)              │ │  │
│  │  └────────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
│         ↓         ↓         ↓           ↓          ↓            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         Database & Storage Layer                        │  │
│  │ - MySQL Database (StatefulSet: 1)                       │  │
│  │ - PersistentVolume (3Gi)                                │  │
│  │ - PersistentVolumeClaim                                 │  │
│  │ - Host Path: /home/alok/data/mysql                      │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │    Configuration & Security                            │  │
│  │ - 5 ConfigMaps (Service-specific configs)              │  │
│  │ - 3 Secret Sets (Credentials & Certificates)           │  │
│  │ - NetworkPolicy (Label-based access control)           │  │
│  │ - RBAC (Role-based access control)                     │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Statistics

| Metric | Value |
|--------|-------|
| **Namespaces** | 4 (home-stack, home-stack-iot, home-stack-db, home-stack-dmz) |
| **Services** | 7 (6 application + 1 database) |
| **Deployments** | 5 (home-api, home-auth, home-search, iot-telemetry, mosquitto) |
| **StatefulSets** | 2 (home-etl, mysql) |
| **Nodes Required** | 2+ (jgte, khbr + flexible) |
| **Storage Size** | 3Gi (MySQL persistent volume) |
| **CPU Request** | ~1.2 cores minimum |
| **Memory Request** | ~1.5Gi minimum |
| **ConfigMaps** | 5 |
| **Secrets** | 3 |
| **PersistentVolumes** | 1 |
| **PersistentVolumeClaims** | 1 |

---

## 🔍 Quick Navigation

### By Use Case

**I want to understand...**

- **How external traffic flows** → Start with K8S_ARCHITECTURE_OVERVIEW.md
- **Service dependencies** → K8S_APPLICATION_LAYERS.md > Service Dependencies Map
- **Pod-to-pod communication** → K8S_DEPLOYMENT_DATAFLOW.md > Service Communication Map
- **Database connectivity** → K8S_COMPONENT_DIAGRAM.md > Database Layer
- **How scaling works** → K8S_ARCHITECTURE_OVERVIEW.md > Scaling Strategy
- **Health check behavior** → K8S_APPLICATION_LAYERS.md > Pod Lifecycle
- **Configuration injection** → K8S_COMPONENT_DIAGRAM.md > Pod Specifications
- **Security & isolation** → K8S_DEPLOYMENT_DATAFLOW.md > Network Security

### By Service

**I want to know about...**

| Service | File | Section |
|---------|------|---------|
| home-api-service | K8S_SERVICES_SPECIFICATIONS.md | Section 1 |
| home-auth-service | K8S_SERVICES_SPECIFICATIONS.md | Section 2 |
| home-search-service | K8S_SERVICES_SPECIFICATIONS.md | Section 3 |
| home-etl-service | K8S_SERVICES_SPECIFICATIONS.md | Section 4 |
| iot-telemetry-service | K8S_SERVICES_SPECIFICATIONS.md | Section 5 |
| mosquitto-service | K8S_SERVICES_SPECIFICATIONS.md | Section 6 |
| mysql | K8S_SERVICES_SPECIFICATIONS.md | Section 7 |

---

## 🚀 Key Design Patterns

### 1. Microservices Architecture
- Each service has a dedicated namespace/logical boundary
- Independent deployment and scaling
- Service-to-service communication via ClusterIP

### 2. Horizontal Pod Autoscaling (HPA)
Services that auto-scale:
- home-api-service
- home-auth-service  
- home-search-service
- iot-telemetry-service

Services that don't auto-scale:
- home-etl-service (batch processing - needs consistency)
- mosquitto-service (message broker - state management)
- mysql (database - stateful)

### 3. Node Affinity Strategy
```
Stateful/Critical:
├── jgte node: MySQL, home-etl
└── khbr node: iot-telemetry, mosquitto

Flexible Scheduling:
├── home-api (no constraint)
├── home-auth (no constraint)
└── home-search (prefer microk8s-worker)
```

### 4. Configuration Management
```
Three-layer approach:
├── ConfigMaps (non-sensitive config)
│   ├── Database URLs
│   ├── Logging levels
│   ├── Service-specific properties
│   └── MQTT broker settings
│
├── Secrets (sensitive data)
│   ├── Database credentials
│   ├── JWT secrets
│   ├── OAuth tokens
│   └── TLS certificates
│
└── Environment variables (runtime config)
    ├── Injected from ConfigMaps
    ├── Injected from Secrets
    └── Pod-specific values (metadata.name)
```

### 5. Health Check Strategy
```
All Spring Boot services implement:
├── Liveness Probe (/actuator/health/liveness)
│   └── Detects stuck/crashed processes
│
└── Readiness Probe (/actuator/health/readiness)
    └── Detects services not ready for traffic

Action on Failure:
├── Liveness: Container restart
└── Readiness: Remove from service LB
```

---

## 📊 Network Topology

### Ingress Rules
```
CloudFlare (hdash.alok.world)
├── / → dashboard-service:80
├── /home/api/actuator → defaultbackend:80
├── /home/etl/actuator → defaultbackend:80
└── /home/auth/actuator → defaultbackend:80

Domain (alok-home.com)
├── /home/api → home-api-service:8081
├── /home/etl → home-etl-service:8081
├── /home/auth → home-auth-service:8081
├── /home/search → home-search-service:8081
└── / → dashboard-service:80

Local (jgte)
└── / → dashboard-service:80
```

### Database Access
```
All services connect to:
├── mysql.home-stack-db.svc.cluster.local:3306
├── Credentials from: mysql-secrets
├── Network Label: network/db-access=true
└── AccessMode: ReadWriteOnce (single pod)
```

---

## 🔐 Security Considerations

1. **Network Policies**
   - Label-based pod-to-database access control
   - Prevents unauthorized database connections

2. **RBAC**
   - Cluster admin users defined
   - Dashboard admin users defined
   - Role-based access enforcement

3. **Secrets Management**
   - Database credentials separated from code
   - TLS certificates for IoT/MQTT
   - OAuth credentials stored securely

4. **Service Isolation**
   - Multi-namespace architecture
   - ClusterIP for internal services
   - NodePort only for required external access (MQTT, MySQL)

---

## 📈 Scaling Considerations

### Horizontal Scaling (HPA)
- Automatic based on CPU/Memory metrics
- Suitable for stateless services
- Enabled for: API, Auth, Search, IoT Telemetry

### Vertical Scaling
- Increase resource limits
- Current memory limit: 512Mi per pod
- Can be increased if needed

### Database Scaling
- Single MySQL instance (StatefulSet: 1)
- Scale up: Increase PV size (requires storage expansion)
- Scale out: Would require MySQL replication setup

---

## 🛠️ Common Operations

### Check Service Status
```bash
kubectl get services -n home-stack
kubectl get services -n home-stack-iot
kubectl get services -n home-stack-db
```

### View Pod Logs
```bash
kubectl logs -n home-stack deployment/home-api-deployment
kubectl logs -n home-stack-iot deployment/iot-telemetry-deployment
```

### Check Health Status
```bash
# Verify readiness
kubectl get pods -n home-stack -o wide

# Check specific health endpoint
kubectl exec -n home-stack <pod-name> -- curl localhost:8081/actuator/health
```

### Monitor HPA
```bash
kubectl get hpa -n home-stack
kubectl describe hpa -n home-stack home-api-hpa
```

### Access Database
```bash
# From within cluster (internal)
mysql -h mysql.home-stack-db.svc.cluster.local -P 3306 -u <user> -p

# From outside cluster (NodePort)
mysql -h <node-ip> -P 32306 -u <user> -p
```

---

## 📝 YAML Files Reference

Located in: `/Users/aloksingh/git/home-stack/yaml/`

### Service Definitions
- `home-api-service.yaml`
- `home-auth-service.yaml`
- `home-search-service.yaml`
- `home-etl-service.yaml`
- `iot-telemetry-service.yaml`
- `iot-mosquitto-service.yaml`
- `mysql-service.yaml`

### Configuration
- `config-map.yaml` (general configuration)
- `secrets.yaml` (credentials and secrets)
- `iot-config-map.yaml` (IoT-specific config)
- `iot-telemetry-config-map.yaml` (telemetry config)

### Infrastructure
- `ingress.yaml` (ingress rules)
- `namespace.yaml` (namespace definitions)
- `networkpolicy.yaml` (network policies)
- `home-hpa.yaml` (auto-scaling configuration)

### Monitoring & Management
- `kubernetes-dashboard.yaml`
- `metrix-server.yaml` (metrics collection)
- `jaeger-all-in-one-template.yml` (distributed tracing)

### Utilities
- `home-nw-tshoot.yaml` (network troubleshooting pod)
- `git-commit-cronjob.yaml` (scheduled git commits)
- `dashboard-service.yaml` (dashboard UI)

---

## 🔄 Data Flow Summary

```
User/IoT Device
    ↓
CloudFlare DNS / Ingress Controller
    ↓
Load Balancer → Ingress Rules
    ↓
Application Services (Auth → API → Database)
    ↓
MySQL Database
    ↓
Search Index / ETL Pipeline
    ↓
Analytics / Reports / Dashboard
```

---

## ✅ Deployment Checklist

- [x] Multi-namespace architecture
- [x] Service mesh-less design (direct communication)
- [x] Health checks (liveness & readiness)
- [x] Resource requests & limits
- [x] Horizontal autoscaling (HPA)
- [x] Persistent storage
- [x] Configuration management (ConfigMaps & Secrets)
- [x] Network policies
- [x] RBAC
- [x] Rolling updates
- [x] Node affinity for critical services
- [x] External access via Ingress and NodePort
- [x] Cross-namespace communication
- [x] Image pull policy (Always)

---

## 📚 Additional Resources

### Related Documentation
- Spring Boot Actuator: Health endpoints configuration
- Kubernetes Documentation: StatefulSets, HPA, Network Policies
- NGINX Ingress: Routing and proxy configuration
- MySQL: Database replication and backup strategies

### Future Improvements
- Service mesh integration (Istio/Linkerd)
- Distributed tracing integration
- Custom metrics for HPA
- Database backup and recovery strategies
- Multi-region deployment
- Load balancing optimization

---

Generated: 2026-08-07
Last Updated: Based on YAML files analysis
Source: `/Users/aloksingh/git/home-stack/yaml/`
