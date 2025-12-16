# Linkding Postgres Char

A collection of Helm charts for deploying Linkding (a bookmark manager) with PostgreSQL backend on Kubernetes bare metal cluster.

## Overview

This repository contains two Helm charts that work together:

- **postgres**: Deploys PostgreSQL database as the backend
- **linkding**: Deploys the linkding web application

## Charts

### Linkding

A Helm chart for deploying Linkding, a self-hosted bookmark manager.

**Features:**
- Deploy linkding web application (sissbruecker/linkding)
- Service account configuration
- Node affinity targeting worker nodes

**Configuration:**
- Service: ClusterIP on port 9090
- Ingress: Enabled by default
- Superuser credentials configurable
- Node affinity for worker nodes

### Postgres

A Helm chart for deploying PostgreSQL database.

**Features:**
- PostgreSQL 14 deployment
- Persistent storage with hostPath
- Configurable database credentials
- Service account configuration
- Node affinity targeting worker nodes

**Configuration:**
- Service: NodePort on port 5432
- Database: linkding
- Persistence: 2Gi with hostPath mounting
- Node affinity for worker nodes

## Structure

```
.
├── charts/
│   ├── linkding/           # linkding application chart
│   │   ├── templates/      # Kubernetes templates
│   │   ├── Chart.yaml      # Chart metadata
│   │   └── values.yaml     # Default values
│   └── postgres/           # PostgreSQL database chart
│       ├── templates/      # Kubernetes templates
│       ├── Chart.yaml      # Chart metadata
│       └── values.yaml     # Default values
└── README.md               # This file
```

## Installation

### Prerequisites
- Kubernetes cluster
- Helm 3+
- Appropriate permissions to deploy resources

**IMPORTANT:** The PostgreSQL helm chart must be deployed first before the linkding application, as linkding depends on the database being available.

### Deploy PostgreSQL

The PostgreSQL chart includes persistent data storage using hostPath mounting, ensuring your bookmark data is preserved across pod restarts and deployments.

```bash
helm install postgres ./charts/postgres
```

### Deploy Linkding

```bash
helm install linkding ./charts/linkding
```

### Custom values

You can override default values by creating a custom values file:

```bash
helm install linkding ./charts/linkding -f custom-values.yaml
```

## Configuration

Both charts support extensive configuration through their respective `values.yaml` files. Key configurable parameters include:

- Resource limits and requests
- Image versions and repositories
- Storage settings
- Networking configuration
- Security contexts
- Autoscaling settings

## Node Affinity

Both charts are configured with node affinity to target worker nodes:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: node-type
              operator: In
              values:
                - worker
```

## Version Information

- Linkding image: sissbruecker/linkding:latest
- PostgreSQL image: postgres:14

## Support

This is a Helm chart repository for deploying Linkding with PostgreSQL. For application-specific issues, refer to the respective upstream projects.