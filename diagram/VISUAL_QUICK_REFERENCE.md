# Kubernetes Architecture - Visual Quick Reference

## 🎯 System at a Glance

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER OVERVIEW                         │
│                                                                         │
│  External:  hdash.alok.world ──┐                                      │
│                                 ├──> NGINX Ingress Controller        │
│  External:  alok-home.com ──────┤    (Path-based routing)            │
│                                 └──> Dashboard Service                │
│  Local:     jgte ────────────────┘    (Web UI)                        │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ INGRESS ROUTING LOGIC                                            │ │
│  ├───────────────────────────────────────────────────────────────────┤ │
│  │ Path: /home/api        → home-api-service:8081                  │ │
│  │ Path: /home/auth       → home-auth-service:8081                │ │
│  │ Path: /home/search     → home-search-service:8081              │ │
│  │ Path: /home/etl        → home-etl-service:8081                 │ │
│  │ Path: /iot/telemetry   → iot-telemetry-service:8081            │ │
│  │ Path: /                → dashboard-service:80                  │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌─────────────────────────┐  ┌──────────────────────┐               │
│  │ home-stack Namespace    │  │ home-stack-iot NS    │               │
│  ├─────────────────────────┤  ├──────────────────────┤               │
│  │ ✓ home-api (HPA)        │  │ ✓ iot-telemetry(HPA)│               │
│  │ ✓ home-auth (HPA)       │  │ ✓ mosquitto (1)     │               │
│  │ ✓ home-search (HPA)     │  │                      │               │
│  │ ✓ home-etl (1)          │  │ NodePort: 31883      │               │
│  └─────────────────────────┘  └──────────────────────┘               │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │ All Services → MySQL Database (home-stack-db)                  │ │
│  │              → 3Gi Persistent Volume @ /home/alok/data/mysql   │ │
│  │              → NodePort: 32306 (external)                      │ │
│  └──────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐ │
│  │ CONFIG & SECRETS                                               │ │
│  ├──────────────────────────────────────────────────────────────────┤ │
│  │ ConfigMaps: 5 (home-api, home-auth, home-email, iot-telemetry)│ │
│  │ Secrets:    3 (mysql-secrets, iot, mosquitto)                 │ │
│  │ Policies:   Network policies, RBAC                            │ │
│  └──────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

## 📊 Service Matrix

```
┌──────────────────┬──────────┬────────┬──────┬────────────┬─────────────┐
│ Service          │ Port     │ Type   │ Mode │ Scaling    │ Location    │
├──────────────────┼──────────┼────────┼──────┼────────────┼─────────────┤
│ home-api         │ 8081     │ DEPLOY │ API  │ HPA (2-N)  │ Flexible    │
│ home-auth        │ 8081     │ DEPLOY │ AUTH │ HPA (2-N)  │ Flexible    │
│ home-search      │ 8081     │ DEPLOY │ API  │ HPA (1-N)  │ Worker      │
│ home-etl         │ 8081     │ SSFSET │ ETL  │ 1 Fixed    │ jgte        │
│ iot-telemetry    │ 8081     │ DEPLOY │ IOT  │ HPA (1-N)  │ khbr        │
│ mosquitto        │ 1883     │ DEPLOY │ MSG  │ 1 Fixed    │ khbr        │
│ mysql            │ 3306     │ SSFSET │ DB   │ 1 Fixed    │ jgte        │
└──────────────────┴──────────┴────────┴──────┴────────────┴─────────────┘
Legend: DEPLOY=Deployment, SSFSET=StatefulSet, HPA=Auto-scale
```

## 🔄 Service Communication

```
                    ┌─────────────┐
                    │  Google     │
                    │  OAuth      │
                    └──────┬──────┘
                           │
                    ┌──────▼──────────────┐
                    │  Ingress Controller │
                    │  (NGINX)            │
                    └──────┬──────────────┘
                    ┌──────┴──────┐
        ┌───────────┼───────────┬─┴────────┐
        │           │           │          │
        ▼           ▼           ▼          ▼
     home-api   home-auth  home-search  home-etl
        │           │           │          │
        └────────┬──┼───────────┘          │
                 │  │                      │
                 ▼  ▼                      │
            ┌─────────────┐                │
            │   MySQL     │◄───────────────┤
            │  Database   │                │
            └─────────────┘                │
                                          │
                         ┌─────────────────┴──┐
                         │   iot-telemetry   │
                         │                   │
                         │  mosquitto        │
                         └───────────────────┘
```

## 🗂️ Namespace Layout

```
┌────────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                              │
│                                                                    │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐  │
│  │ home-stack       │  │ home-stack-iot   │  │ home-stack-db  │  │
│  │                  │  │                  │  │                │  │
│  │ • home-api       │  │ • iot-telemetry  │  │ • mysql        │  │
│  │ • home-auth      │  │ • mosquitto      │  │ • PV/PVC       │  │
│  │ • home-search    │  │ • ConfigMap      │  │ • Storage      │  │
│  │ • home-etl       │  │ • Secrets        │  │                │  │
│  │ • ConfigMap      │  │                  │  └────────────────┘  │
│  │ • Secrets        │  └──────────────────┘                       │
│  └──────────────────┘                                             │
│                                                                    │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │ home-stack-dmz (DMZ Zone)                                  │  │
│  │                                                             │  │
│  │ • Ingress Rules (3)                                        │  │
│  │ • RBAC (cluster-admin, dashboard-admin)                    │  │
│  │ • NetworkPolicy                                            │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

## 🎯 Data Flow Path

```
User Request
    │
    ▼
┌──────────────────────┐
│   CloudFlare/DNS     │
│  hdash.alok.world    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│     Load Balancer    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────┐
│   NGINX Ingress Controller   │
│   (Path-based routing)       │
└──────────┬────────┬──────┬───────────┘
           │        │      │
    ┌──────▼──┐ ┌───▼────┐ ┌──▼────────┐
    │  API    │ │  AUTH  │ │  SEARCH   │
    │ :8081   │ │ :8081  │ │  :8081    │
    └─────┬───┘ └──┬─────┘ └──┬───────┘
          │        │          │
          └────────┼──────────┘
                   │
                   ▼
          ┌─────────────────┐
          │   MySQL DB      │
          │    :3306        │
          └─────────────────┘

Query Results/Data → Response → User
```

## ⚙️ Pod Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│ POD STARTUP SEQUENCE                                        │
├─────────────────────────────────────────────────────────────┤
│ 1. Container Started (Image Pulled - Always)                │
│ 2. Wait: initialDelaySeconds (5-90s per service)            │
│ 3. ✓ Liveness Probe (Can container survive?)               │
│    └─ Endpoint: /actuator/health/liveness                  │
│ 4. ✓ Readiness Probe (Ready for traffic?)                  │
│    └─ Endpoint: /actuator/health/readiness                 │
│ 5. Pod Added to Service LB (Routes traffic)                │
│ 6. Continuous Health Checks (Every 10-60s)                 │
│ 7. If Liveness Fails → Container Restart                   │
│ 8. If Readiness Fails → Remove from LB                     │
└─────────────────────────────────────────────────────────────┘
```

## 💾 Storage Architecture

```
┌─────────────────────────────────────────────────────┐
│ Stateful Storage Setup                              │
│                                                     │
│  ┌──────────────────┐                              │
│  │  MySQL Pod       │                              │
│  │  (Container)     │                              │
│  └────────┬─────────┘                              │
│           │                                        │
│           │ mounts                                 │
│           ▼                                        │
│  ┌──────────────────────────┐                      │
│  │ PersistentVolumeClaim    │                      │
│  │ mysql-pv-claim (3Gi)     │                      │
│  └────────┬─────────────────┘                      │
│           │                                        │
│           │ binds 1:1                              │
│           ▼                                        │
│  ┌──────────────────────────────┐                  │
│  │ PersistentVolume             │                  │
│  │ mysql-pv-volume (3Gi)        │                  │
│  │ storageClass: manual         │                  │
│  │ accessMode: ReadWriteOnce    │                  │
│  └────────┬─────────────────────┘                  │
│           │                                        │
│           │ hostPath                               │
│           ▼                                        │
│  ┌──────────────────────────────────┐              │
│  │ Host Filesystem                  │              │
│  │ /home/alok/data/mysql            │              │
│  │ (Node: jgte)                     │              │
│  └──────────────────────────────────┘              │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## 🔐 Security Layers

```
┌──────────────────────────────────────────────────────┐
│ SECURITY ARCHITECTURE                               │
├──────────────────────────────────────────────────────┤
│                                                      │
│ 1. INGRESS LAYER                                    │
│    └─ HTTPS/TLS termination                        │
│    └─ Path-based access control                    │
│                                                      │
│ 2. AUTHENTICATION LAYER                             │
│    └─ OAuth 2.0 (Google)                           │
│    └─ JWT Token validation                         │
│    └─ Session management                           │
│                                                      │
│ 3. NETWORK POLICIES                                 │
│    └─ Label-based: network/db-access=true          │
│    └─ Pod-to-database access control               │
│                                                      │
│ 4. RBAC (Role-Based Access Control)                │
│    └─ cluster-admin-user                           │
│    └─ dashboard-admin-user                         │
│                                                      │
│ 5. SECRETS MANAGEMENT                               │
│    ├─ mysql-secrets (DB credentials)               │
│    ├─ iot-telemetry-secret (JKS/certs)             │
│    └─ mosquitto-secret (TLS/CA/ACL)                │
│                                                      │
│ 6. IMAGE PULL POLICY                               │
│    └─ Always (forces latest image check)           │
│                                                      │
└──────────────────────────────────────────────────────┘
```

## 📈 Resource Allocation

```
┌─────────────────────────────────────────────────────┐
│ CLUSTER RESOURCE SUMMARY                            │
├─────────────────────────────────────────────────────┤
│                                                     │
│ CPU ALLOCATION (at minimum replicas):               │
│ ┌──────────────────────────────────────────────┐   │
│ │ home-api:        200m × 2 = 400m            │   │
│ │ home-auth:       100m × 2 = 200m            │   │
│ │ home-search:     200m × 1 = 200m            │   │
│ │ iot-telemetry:   200m × 1 = 200m            │   │
│ │ ────────────────────────────────────────── │   │
│ │ TOTAL REQUEST:            ~1.2 cores       │   │
│ └──────────────────────────────────────────────┘   │
│                                                     │
│ MEMORY ALLOCATION (at minimum replicas):            │
│ ┌──────────────────────────────────────────────┐   │
│ │ home-api:        256Mi × 2 = 512Mi          │   │
│ │ home-auth:       256Mi × 2 = 512Mi          │   │
│ │ home-search:     256Mi × 1 = 256Mi          │   │
│ │ iot-telemetry:   256Mi × 1 = 256Mi          │   │
│ │ ────────────────────────────────────────── │   │
│ │ REQUEST TOTAL:            ~1.5Gi           │   │
│ │ LIMIT TOTAL (512Mi each):  ~3Gi             │   │
│ └──────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## 🚀 Deployment Characteristics

```
┌────────────────────────────────────────────────────────┐
│ DEPLOYMENT STRATEGIES                                 │
├────────────────────────────────────────────────────────┤
│                                                        │
│ ROLLING UPDATES (Deployment Services):                │
│ • One pod replaced at a time                          │
│ • Old pods terminate after new ready                  │
│ • Zero-downtime deployments                           │
│ • Applied to: api, auth, search, iot-telemetry      │
│                                                        │
│ FIXED REPLICAS (StatefulSet Services):                │
│ • home-etl: 1 (batch processing - needs stability)   │
│ • mysql: 1 (database - persistent state)             │
│ • mosquitto: 1 (message broker)                      │
│                                                        │
│ AUTO-SCALING (HPA):                                   │
│ • Monitors CPU & Memory metrics                      │
│ • Scales based on configured thresholds              │
│ • Target services: api, auth, search, iot-telemetry │
│                                                        │
└────────────────────────────────────────────────────────┘
```

## 🔗 External Access Points

```
┌──────────────────────────────────────────────┐
│ HOW TO ACCESS SERVICES                       │
├──────────────────────────────────────────────┤
│                                              │
│ INTERNAL (Within Cluster):                   │
│ • mysql.home-stack-db:3306                  │
│ • home-api-service.home-stack:8081          │
│ • home-auth-service.home-stack:8081         │
│ • home-search-service.home-stack:8081       │
│ • iot-telemetry-service.home-stack-iot:8081│
│ • mosquitto-service.home-stack-iot:1883     │
│                                              │
│ EXTERNAL (From Outside Cluster):            │
│ • HTTP/HTTPS: hdash.alok.world (443)        │
│ • HTTP/HTTPS: alok-home.com (443)           │
│ • MQTT: <node-ip>:31883 (mosquitto)         │
│ • MySQL: <node-ip>:32306 (database)         │
│                                              │
│ LOCAL TESTING:                               │
│ • HTTP: jgte:80                             │
│                                              │
└──────────────────────────────────────────────┘
```

## 📋 Quick Checklists

### Deployment Readiness
- ✅ All namespaces created
- ✅ ConfigMaps deployed
- ✅ Secrets configured
- ✅ PV/PVC provisioned
- ✅ Services deployed
- ✅ Pods running and healthy
- ✅ Ingress rules active
- ✅ Health checks passing

### Security Checklist
- ✅ Network policies enforced
- ✅ Secrets not in code
- ✅ RBAC configured
- ✅ TLS certificates valid
- ✅ OAuth provider configured
- ✅ Database credentials secure

### Monitoring Checklist
- ✅ Health endpoints responding
- ✅ Metrics server running
- ✅ HPA metrics collecting
- ✅ Pod logs accessible
- ✅ Events logging
- ✅ Readiness probes passing

---

## 📚 Document Locations

All documentation files are located in:
`/Users/aloksingh/.copilot/session-state/b860028f-aae2-449e-afc4-6eb6353ce8ee/files/`

- **README.md** - Start here for overview
- **K8S_ARCHITECTURE_OVERVIEW.md** - System design & namespaces
- **K8S_COMPONENT_DIAGRAM.md** - Detailed pod architecture
- **K8S_DEPLOYMENT_DATAFLOW.md** - Data flows & communication
- **K8S_APPLICATION_LAYERS.md** - 7-tier application stack
- **K8S_SERVICES_SPECIFICATIONS.md** - Detailed service specs
- **VISUAL_QUICK_REFERENCE.md** - This file

---

**Created**: 2026-08-07  
**Based on YAML Files**: /Users/aloksingh/git/home-stack/yaml/  
**Architecture Analysis**: Complete K8s cluster review
