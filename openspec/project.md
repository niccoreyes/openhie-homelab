# Project Context

## Purpose
GitOps-based homelab project deploying OpenHIE (Open Health Information Exchange) components on a Raspberry Pi 8gb RAM K3s cluster. The project demonstrates healthcare interoperability infrastructure-as-code practices using Flux v2 for automated deployment and management of Kubernetes resources. Designed for educational and development purposes in resource-constrained environments.

## Tech Stack
- **Kubernetes Distribution**: K3s (lightweight Kubernetes for edge/IoT)
- **GitOps Tool**: Flux v2 (manifest generation + reconciliation)
- **Cluster Management**: Single-node Raspberry Pi cluster (pi-cluster)
- **Version Control**: Git (GitHub repository as source of truth)
- **VPN/Access**: Tailscale operator for secure remote access
- **Container Management**: Portainer for Kubernetes UI and cluster administration
- **Healthcare API**: HAPI FHIR Server (FHIR R4 standards-based API)
- **Interoperability Layer**: OpenHIM (message routing and transformation)
- **Terminology Management**: OCL (Open Concept Lab)

## Project Conventions

### Code Style
- Kubernetes manifests in YAML format with consistent indentation
- Resource naming follows Kubernetes conventions (kebab-case)
- Flux kustomizations use conventional structure:
  - `/clusters/pi-cluster/flux-system/apps/` for application deployments
  - `/clusters/pi-cluster/flux-system/openhie-apps/` for healthcare-specific apps
- Secret management via Kubernetes native mechanisms (no plaintext secrets)

### Architecture Patterns
- **GitOps Pattern**: All infrastructure defined in Git, Flux handles reconciliation
- **Single-Cluster Architecture**: Single-node K3s cluster on Raspberry Pi
- **ARM64 Compatibility**: All container images tagged for ARM64 architecture
- **Embedded Databases**: Uses embedded PostgreSQL (HAPI FHIR) and Bitnami MongoDB (OpenHIM)
- **Local Path Storage**: K3s default local-path provisioner for persistent volumes
- **Helm via Flux**: HelmReleases managed through Flux rather than direct Helm usage

### Testing Strategy
No testing strategy needed for now.

### Git Workflow
- **Feature Branch Workflow**: Work directly on main branch for now. But there are future plans to do proper work in feature branches, then merge to main.
- **Commit Convention**: `feat(service): description` format using conventional commits
- **Branch Strategy**: Main branch as primary/production branch
- **GitOps Synchronization**: Flux reconciles from main branch every 10 minutes
- **Commit Prefixes**: `feat` (new features), `fix` (bug fixes), `chore` (maintenance)

## Domain Context

### Healthcare Interoperability Focus
This project implements the **OpenHIE (Open Health Information Exchange) architecture**, a reference implementation for healthcare data exchange in resource-constrained settings. Key domain concepts:

- **FHIR (Fast Healthcare Interoperability Resources)**: Modern standard for healthcare data exchange
- **HAPI FHIR**: Java-based FHIR server implementing FHIR R4 specification
- **OpenHIM**: Middleware component for routing/transforming healthcare messages
- **OCL**: Medical terminology and concept dictionary management
- **Interoperability Layer**: Manages communication between disparate health systems

### Edge Deployment Characteristics
- Designed for **Raspberry Pi K3s clusters** (ARM64 architecture)
- **Lightweight** deployment suitable for low-resource environments
- **Offline-capable** deployment for regions with limited connectivity
- **Development and educational** use case for health informatics training

## Important Constraints

### Technical Constraints
- **ARM64 Architecture**: All container images must be ARM64-compatible
- **Resource Constraints**: Suitable for Raspberry Pi hardware (limited CPU/memory)
- **Single-Node Cluster**: No high availability or multi-node distribution
- **Local Storage Only**: Uses K3s local-path provisioner (no external storage)
- **Network Constraints**: Tailscale VPN for secure remote access (no public ingress by default)

### Deployment Constraints
- **GitOps-Only**: All changes must go through Git repository, no manual kubectl operations
- **Flux Managed**: Applications deployed exclusively through Flux kustomizations
- **No Environment Branches**: Single main branch deployment (no dev/staging/prod differentiation)
- **Embedded Databases**: No external database services (all data in-cluster)

## External Dependencies

### External Services
- **GitHub**: Repository hosting at `git@github.com:niccoreyes/openhie-homelab`
- **Tailscale**: VPN service for secure cluster access (authentication required)
- **Container Registries**:
  - Docker Hub (various images)
  - GHCR (GitHub Container Registry)
  - opensrp/community-charts (OpenHIE community charts)
  - chgl/charts (**DEPRECATED** HAPI FHIR charts - migration in progress to official HAPI FHIR charts)

### Data Sources
- **Helm Repositories**:
  - `https://opensrp.github.io/community-charts` (OpenHIM, HAPI FHIR, OCL)
  - `https://chgl.github.io/charts` (**DEPRECATED**: HAPI FHIR - Migrate to https://github.com/hapifhir/hapi-fhir-jpaserver-starter/tree/master/charts/hapi-fhir-jpaserver)
  - `https://tailscale.com/helm-charts` (Tailscale operator)

### Deprecation Notice
⚠ **Important**: The CHGL HAPI FHIR charts are deprecated in favor of the official HAPI FHIR JPAServer charts at https://github.com/hapifhir/hapi-fhir-jpaserver-starter/tree/master/charts/hapi-fhir-jpaserver. Migration to the official charts is recommended.

### Network Access
- **Tailscale Network**: Required for remote cluster access and management
- **No Public Ingress**: Services exposed locally on the Raspberry Pi (unless Tailscale enabled)

### Server Access
- **Server Logs**: The server is accessible on pi@raspberrypi if on SSH. Tailscale deals with the rest.
- **Kubernetes / Flux Commands**: Run the commands through the mentioned endpoint not from local if needed.
