#  OrbitalStack DevOps Assignment

##  Overview

This project implements a production-grade Kubernetes platform for an IoT device management system (**OrbitalStack**) using:

* Helm (application deployment)
* Kubernetes (orchestration)
* GitOps-ready structure
* Scalable and secure microservices architecture

---



##  High-Level Architecture

```
                +----------------------+
                |     Ingress (NGINX)  |
                +----------+-----------+
                           |
                           v
                   +---------------+
                   |  device-api   |
                   +-------+-------+
                           |
            +--------------+--------------+
            |                             |
            v                             v
+---------------------+       +----------------------+
| telemetry-ingestor  |       |   ota-controller     |
+----------+----------+       +----------+-----------+
           |                             |
           v                             v
      +----------+               +--------------+
      | Timescale |               | Firmware S3 |
      +----------+               +--------------+

                 ^
                 |
         +---------------+
         | mqtt-broker  |
         +---------------+
```

---

##  Helm Chart Structure

Each service has its own Helm chart:

```
helm/charts/
├── device-api/
├── telemetry-ingestor/
├── mqtt-broker/
└── ota-controller/
```

Each chart includes:

* Deployment
* Service
* HPA (if applicable)
* PDB
* ConfigMap
* NetworkPolicy
* ServiceAccount
* ServiceMonitor (conditional)
* ExternalSecret (conditional)

---

##  Key Design Decisions

### 1. OTA Controller Safety

* Uses **Recreate strategy**
* Ensures only ONE instance runs
* Prevents split-brain firmware rollout issues

---

### 2. Resource Management

* All services define **requests & limits**
* Prevents cluster-wide OOM failures

---

### 3. Network Security

* NetworkPolicies implemented
* Default deny (relaxed for local testing)

---

### 4. Scalability

* telemetry-ingestor → HPA enabled
* device-api → HPA enabled
* mqtt-broker → no scaling (stateful)
* ota controller → single instance
* ota controller uses Recreate strategy + leader election to prevent split-brain issues during firmware rollout.

---

### 5. Environment Separation

* values.yaml → local
* values-staging.yaml → staging
* values-production.yaml → production

---

## SOP for Local Setup

### 1. Create cluster

```
kind create cluster
```

### 2. Install services

```
helm install device-api helm/charts/device-api
helm install telemetry-ingestor helm/charts/telemetry-ingestor
helm install mqtt-broker helm/charts/mqtt-broker
helm install ota-controller helm/charts/ota-controller
```

### 3. Verify

```
kubectl get pods
```

---

## LOcal hacks

* NGINX is used as a placeholder container
* Health checks use `/` instead of `/health`
* ExternalSecret and ServiceMonitor are disabled locally

---
## Thank You
