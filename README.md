# Aurora Cloud GitOps 
This repository defines the GitOps source of truth for Aurora's Kubernetes runtime, including platform components (Kubernetes, Istio), shared infrastructure dependencies, and application workloads. It is designed to clearly separate **local development**, **cluster-level platform concerns**, and **production-grade application delivery**.

## Design Principles 
### Strict separation of concerns 
- Source code repos handle build & CI 
- This repo handles deployment state only 

### Environment isolation 
- `dev` -> local dependency simulation / port-forward friendly 
- `prod` -> real cluster services using `svc.cluster.local`

### Kustomize-first 
- No Helm templating logic in apps 
- Git diff = actual runtime diff 

### Istio-native 
- Ingress, traffic routing, security live in GitOps 
- No legacy API Gateway module 


## Repository Layout 
```
aurora-cloud-gitops
├── README.md
│
├── clusters
│ ├── dev-kind
│ │ ├── kustomization.yaml
│ │ ├── namespace.yaml
│ │ └── apps
│ │ └── aurora
│ │ └── kustomization.yaml
│ │
│ └── prod
│ ├── kustomization.yaml
│ ├── namespace.yaml
│ └── apps
│ └── aurora
│ └── kustomization.yaml
│
├── platform
│ ├── k8s
│ │ └── namespaces.yaml
│ │
│ ├── istio
│ │ ├── base
│ │ │ ├── istio-base.yaml
│ │ │ └── kustomization.yaml
│ │ ├── ingress
│ │ │ ├── gateway.yaml
│ │ │ ├── virtualservice.yaml
│ │ │ └── kustomization.yaml
│ │ └── overlays
│ │ └── prod
│ │ └── kustomization.yaml
│ │
│ └── security
│ └── oauth2-proxy
│ ├── deployment.yaml
│ ├── service.yaml
│ └── kustomization.yaml
│
├── infra
│ ├── local-deps
│ │ ├── base
│ │ │ ├── postgres.yaml
│ │ │ ├── rabbitmq.yaml
│ │ │ └── kustomization.yaml
│ │ └── overlays
│ │ └── dev
│ │ ├── namespace.yaml
│ │ ├── patch-postgres.yaml
│ │ ├── patch-rabbitmq.yaml
│ │ └── kustomization.yaml
│ │
│ └── prod-deps
│ ├── postgres
│ │ ├── statefulset.yaml
│ │ ├── service.yaml
│ │ └── kustomization.yaml
│ └── rabbitmq
│ ├── deployment.yaml
│ ├── service.yaml
│ └── kustomization.yaml
│
└── apps
└── aurora
├── base
│ ├── deployment.yaml
│ ├── service.yaml
│ ├── configmap.yaml
│ └── kustomization.yaml
│
└── patch-env.yaml
```

## How Each Layer Is Used 
### clusters/
Entry point for GitOps controllers (Argo CD / Flux).
- Defines what is installed in each cluster
- Never reused across environments 

Example responsibility:
- namespaces
- which apps are deployed 
- which overlays are active 

### platform/
Cluster-wide shared capabilities
Includes: 
- Istio control plane 
- IngressGateway 
- AuthN/AuthZ (OAuth2 Proxy / Keycloak later)

This replaces the legacy **apigw module** completely.


### infa/
Runtime depdnencies
**local-deps**
- Used for **developer experience only**
- Port-forward friendly 
- No Aurora application images 
- CI pipelines may depend on this 

**prod-deps**
- Real backing services 
- Stable DNS: `*.svc.cluster.local`
- Used only in prod clusters 

### apps/aurora
Application deployment manifests.
- `base` -> image, ports, probes
- `overlays/dev` -> localhost / mock friendly 
- `overlays/prod` -> real service discovery 


Example: 
- `application.yml` -> dev 
- `application-prod.yml` -> prod 


## CI vs GitOps Boundary 
### Stage: CI 
- build image, run tests, push image 

### Stage: GitOps 
- declare which image version deploys and runs


CI updates image tags or kustomize patches, not clusters directly 


### Typical Workflow 
- Developer runs local Aurora services 
- Uses `infra/local-deps` via port-forward 
- CI builds Aurora image 
- GitOps repo updated with new image tag 
- Argo CD syncs prod cluster 


### Why This Structure Works Well 
- Clean mental model 
- No coupling between CI and runtime 
- Easy Istio traffic experiments 
- Easy Git diff & rollback 


## LICENSE 
- [LICENSE](./LICENSE)