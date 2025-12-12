# OpenHIE Homelab: GitOps Healthcare Interoperability on Raspberry Pi K3s

[![GitOps](https://img.shields.io/badge/GitOps-Flux%20v2-blue)](https://fluxcd.io)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-K3s%20on%20Raspberry%20Pi-326CE5)](https://k3s.io)
[![Healthcare](https://img.shields.io/badge/Healthcare-OpenHIE%20FHIR%20R4-008080)](https://ohie.org)
[![Architecture](https://img.shields.io/badge/Architecture-ARM64%20Edge%20Deployment-00979D)](https://www.arm.com)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A GitOps-based homelab project deploying **OpenHIE (Open Health Information Exchange)** components on a Raspberry Pi K3s cluster using Flux v2. Designed for junior developers and healthcare professionals exploring healthcare interoperability infrastructure in resource-constrained environments.

## What is OpenHIE?

OpenHIE (Open Health Information Exchange) is an open-source framework for healthcare data exchange in resource-constrained settings. It provides a reference architecture for connecting disparate health information systems using modern standards like **FHIR (Fast Healthcare Interoperability Resources)**.

This project implements key OpenHIE components in a homelab environment, perfect for learning about:
- Healthcare data interoperability
- FHIR APIs and clinical data repositories
- GitOps and infrastructure-as-code
- Edge computing on Raspberry Pi

## Project Goals

- **Educational**: Learn healthcare IT infrastructure through hands-on deployment
- **Accessible**: Run on affordable Raspberry Pi hardware (8GB RAM)
- **GitOps-Driven**: All infrastructure defined as code with automated deployment
- **Real-World Components**: Deploy production-grade healthcare systems (HAPI FHIR, OpenHIM, OCL)
- **Secure Remote Access**: Use Tailscale VPN for secure homelab access

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Repository                        │
│              (Source of Truth for GitOps)                   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                    Raspberry Pi Cluster                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                     K3s Kubernetes                  │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │    │
│  │  │   Flux v2   │  │  Tailscale  │  │  Portainer  │  │    │
│  │  │  (GitOps)   │  │   Operator  │  │    (UI)     │  │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │    │
│  │                                                     │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐ │    │
│  │  │  HAPI FHIR  │  │   OpenHIM   │  │     OCL      │ │    │
│  │  │  (CDR)      │  │ (Router)    │  │ (Terminology)│ │    │
│  │  └─────────────┘  └─────────────┘  └──────────────┘ │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Purpose | Technology |
|-----------|---------|------------|
| **K3s** | Lightweight Kubernetes distribution | Kubernetes for ARM64 |
| **Flux v2** | GitOps automation tool | Continuous deployment |
| **HAPI FHIR** | Clinical Data Repository (CDR) | FHIR R4 API server |
| **OpenHIM** | Health Information Mediator | Message routing/transformation |
| **OCL** | Open Concept Lab | Medical terminology management |
| **Tailscale** | Secure VPN access | Zero-config networking |
| **Portainer** | Kubernetes management UI | Container orchestration |

## Prerequisites

### Hardware Requirements
- Raspberry Pi 4/5 with **8GB RAM** (recommended)
- MicroSD card (32GB+)
- Power supply (USB-C, 3A+)
- Network connection (Ethernet recommended)

### Software Requirements
- **Raspberry Pi OS Lite** (64-bit) or Ubuntu Server
- **K3s** (lightweight Kubernetes)
- **Tailscale account** (free tier sufficient)
- **GitHub account** (for repository fork)

### Knowledge Prerequisites
- Basic Linux command line experience
- Understanding of Kubernetes concepts (pods, services, deployments)
- Familiarity with YAML configuration files
- Interest in healthcare IT/interoperability

## Quick Start Guide

### 1. Clone the Repository
```bash
git clone https://github.com/niccoreyes/openhie-homelab.git
cd openhie-homelab
```

### 2. Set Up Your Raspberry Pi K3s Cluster

#### Install K3s on Raspberry Pi:
```bash
# Install K3s (we'll use Tailscale ingress)
curl -sfL https://get.k3s.io | sh

# Get kubeconfig for local access
sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config
```

#### Verify K3s Installation:
```bash
kubectl get nodes
# Should show: raspberrypi   Ready    control-plane,master   5m   v1.27.4+k3s1
```

### 3. Bootstrap Flux CD

#### Install Flux CLI:
```bash
# On your local machine (not Raspberry Pi)
curl -s https://fluxcd.io/install.sh | sh
```

#### Bootstrap Flux with Your GitHub Repository:
```bash
# Replace with your GitHub username and repository
flux bootstrap github \
  --owner=your-github-username \
  --repository=openhie-homelab \
  --branch=main \
  --path=./clusters/pi-cluster \
  --personal
```

### 4. Configure Tailscale for Secure Access

#### Create Tailscale OAuth Secret:
```bash
# Get your Tailscale OAuth client ID and secret from:
# https://login.tailscale.com/admin/settings/oauth

kubectl create secret generic operator-oauth \
  --namespace=tailscale \
  --from-literal=TS_CLIENT_ID=your-client-id \
  --from-literal=TS_CLIENT_SECRET=your-client-secret
```

#### Deploy Tailscale Operator:
The Tailscale operator is already configured in the GitOps manifests and will deploy automatically via Flux.

### 5. Access Your Deployed Services

Once Flux reconciles (takes 5-10 minutes), access services via Tailscale:

| Service | URL | Purpose |
|---------|-----|---------|
| **Portainer** | `https://portainer.tailXXXXX.ts.net` | Kubernetes management UI |
| **HAPI FHIR** | `https://cdr.tailXXXXX.ts.net` | FHIR API server |
| **OpenHIM** | `https://openhim.tailXXXXX.ts.net` | Health information mediator |
| **OCL** | `https://ocl.tailXXXXX.ts.net` | Terminology server |

Replace `tailXXXXX.ts.net` with your Tailscale MagicDNS domain.

## 📁 Project Structure

```
openhie-homelab/
├── clusters/                    # Kubernetes cluster configurations
│   └── pi-cluster/             # Raspberry Pi cluster
│       ├── flux-system/        # Flux CD controllers and configuration
│       ├── infrastructure/     # Infrastructure components (Portainer, Tailscale, etc.)
│       ├── apps/               # Application deployments (HAPI FHIR, OpenHIM, OCL)
│       ├── config/             # Cluster configuration and secrets
│       ├── jobs/               # Job definitions and CronJobs
│       └── repositories/       # Git/Helm repository definitions
├── openspec/                   # Spec-driven development
│   ├── project.md             # Project conventions and context
│   ├── specs/                 # Current specifications
│   └── changes/               # Proposed changes
├── AGENTS.md                  # AI assistant instructions
└── README.md                  # This file
```

## 🔧 Healthcare Components Deep Dive

### HAPI FHIR Server (Clinical Data Repository)
- **Purpose**: Stores and serves healthcare data using FHIR R4 standard
- **Access**: `https://cdr.tailXXXXX.ts.net/fhir/metadata`
- **Features**: 
  - RESTful FHIR API
  - PostgreSQL database (embedded)
  - FHIR resource validation
  - Search capabilities

### OpenHIM (Health Information Mediator)
- **Purpose**: Routes and transforms healthcare messages between systems
- **Access**: `https://openhim.tailXXXXX.ts.net`
- **Features**:
  - Message routing with configurable workflows
  - Transaction logging and auditing
  - Authentication and authorization
  - API gateway functionality

### OCL (Open Concept Lab)
- **Purpose**: Manages medical terminologies and concept dictionaries
- **Access**: `https://ocl.tailXXXXX.ts.net`
- **Features**:
  - Terminology management (ICD-10, SNOMED CT, LOINC, etc.)
  - Concept mapping and relationships
  - Version control for terminologies
  - REST API for terminology services

## 🛠️ Development Workflow

This project uses **OpenSpec** for spec-driven development:

### FluxCD Repository Structure
The project follows FluxCD best practices with a clear separation of concerns:

- **infrastructure/**: Core infrastructure components like Portainer and Tailscale operator
- **apps/**: OpenHIE applications (HAPI FHIR, OpenHIM, OCL)
- **config/**: Cluster configurations and secrets
- **jobs/**: Batch jobs and CronJobs for data processing tasks
- **repositories/**: Git and Helm repository definitions
- **flux-system/**: Flux controllers and sync configuration

### Running Jobs with Flux
The jobs directory supports running batch jobs and CronJobs as part of your GitOps workflow. Create job definitions in the `clusters/pi-cluster/jobs/` directory to implement recurring or one-time tasks such as:
- Healthcare data backups
- Data validation and cleanup
- Report generation
- Data migration tasks

### Making Changes
1. **Explore Current State**:
   ```bash
   openspec list                 # List active changes
   openspec list --specs         # List specifications
   ```

2. **Create a Change Proposal** (for new features/breaking changes):
   ```bash
   # Example: Add new healthcare component
   mkdir -p openspec/changes/add-new-component/
   # Create proposal.md, tasks.md, and spec deltas
   ```

3. **Validate Changes**:
   ```bash
   openspec validate change-name --strict
   ```

4. **Implement Approved Changes**:
   - Update Kubernetes manifests in appropriate directories (`infrastructure/`, `apps/`, `jobs/`, etc.)
   - Let Flux automatically deploy changes
   - Test via Tailscale access

### GitOps Flow
```
Local Changes → Git Commit → GitHub → Flux Reconciliation → Kubernetes Deployment
```

## Testing Your Deployment

### Verify FHIR Server
```bash
# Check FHIR conformance statement
curl -k https://cdr.tailXXXXX.ts.net/fhir/metadata | jq .

# Create a test patient
curl -k -X POST https://cdr.tailXXXXX.ts.net/fhir/Patient \
  -H "Content-Type: application/fhir+json" \
  -d '{
    "resourceType": "Patient",
    "active": true,
    "name": [{"use": "official", "text": "John Doe"}]
  }'
```

### Check Kubernetes Resources
```bash
# List all deployed pods
kubectl get pods --all-namespaces

# Check Flux reconciliation status
flux get kustomizations --all-namespaces

# View Tailscale operator logs
kubectl logs -n tailscale deployment/tailscale-operator
```

## Troubleshooting

### Common Issues

#### 1. Flux Not Reconciling
```bash
# Check Flux logs
flux logs --all-namespaces

# Reconcile manually
flux reconcile kustomization flux-system
```

#### 2. Kustomize Build Failures
This occurs when kustomization.yaml files are malformed, empty, or reference non-existent paths:
```bash
# Check for empty or malformed kustomization.yaml files
find . -name "kustomization.yaml" -exec ls -la {} \;

# Validate kustomization.yaml files have proper resource references
# Example of correct format:
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - resource1.yaml
  - resource2.yaml
```

If a directory has an empty kustomization.yaml file, either populate it with resources or remove the file entirely.

#### 3. Tailscale Access Issues
- Verify Tailscale operator is running: `kubectl get pods -n tailscale`
- Check OAuth secret is configured correctly
- Ensure your device is connected to Tailscale network

#### 4. Resource Constraints on Raspberry Pi
```bash
# Check resource usage
kubectl top nodes
kubectl top pods --all-namespaces

# Adjust resource limits in manifests if needed
```

#### 5. HAPI FHIR Not Starting
- Check PostgreSQL pod is running
- Verify persistent volume claims
- Review HAPI FHIR logs: `kubectl logs deployment/hapi-fhir-jpaserver`

### Debugging Commands
```bash
# Get detailed information about a failing pod
kubectl describe pod <pod-name>

# View container logs
kubectl logs <pod-name> --tail=50

# Check events in namespace
kubectl get events --sort-by='.lastTimestamp'
```

## Learning Resources

### Healthcare Interoperability
- [OpenHIE Architecture](https://ohie.org/architecture/)
- [FHIR Specification](https://www.hl7.org/fhir/)
- [HAPI FHIR Documentation](https://hapifhir.io/hapi-fhir/docs/)
- [OpenHIM Documentation](https://openhim.org/docs/)

### Kubernetes & GitOps
- [K3s Documentation](https://docs.k3s.io/)
- [Flux CD Documentation](https://fluxcd.io/flux/components/)
- [Tailscale Kubernetes Operator](https://tailscale.com/kb/1234/kubernetes-operator/)
- [Portainer for Kubernetes](https://docs.portainer.io/start/install/kubernetes)

### Raspberry Pi & Edge Computing
- [K3s on Raspberry Pi Guide](https://docs.k3s.io/installation/arm64)
- [Raspberry Pi K3s Cluster](https://github.com/rancher/k3s-ansible)
- [ARM64 Container Images](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/)

## 🤝 Contributing

Contributions are welcome! This is a hobby/educational project meant for:

1. **Developer study**: Learn Kubernetes, GitOps, and healthcare APIs
2. **Healthcare Professionals**: Explore health IT infrastructure
3. **Students**: Study healthcare interoperability patterns

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

### Areas for Contribution
- Additional OpenHIE components
- Improved documentation
- ARM64 optimization
- Monitoring and observability
- Security enhancements

## ⚠️ Important Notes

### This is a Hobby-Grade Deployment
- **Not for production use** with real patient data
- **No HIPAA compliance** - educational purposes only
- **Resource-constrained** - designed for Raspberry Pi limitations
- **Experimental** - components may change or break

### Technical Notes
- **ARM64 Compatibility**: All container images are ARM64-compatible for Raspberry Pi
- **GitOps-Only**: All changes must go through Git repository (no manual kubectl operations)
- **Single-Node Cluster**: No high availability or multi-node distribution
- **Local Storage**: Uses K3s local-path provisioner (no external storage)
- **Chart Deprecation**: The project currently uses CHGL HAPI FHIR charts which are deprecated. Migration to official HAPI FHIR JPAServer charts is recommended.

### Security Considerations
- Tailscale provides secure VPN access
- Default passwords should be changed for production use
- Regular security updates recommended
- Isolate from production networks

### Maintenance
- Regular updates to Kubernetes manifests recommended
- Monitor resource usage on Raspberry Pi
- Backup important data and configurations

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [OpenHIE Community](https://ohie.org) for healthcare interoperability framework
- [Flux CD](https://fluxcd.io) for GitOps automation
- [Tailscale](https://tailscale.com) for secure networking
- [Raspberry Pi Foundation](https://www.raspberrypi.org) for affordable hardware
- [K3s](https://k3s.io) for lightweight Kubernetes

## Support

- **Issues**: Use [GitHub Issues](https://github.com/niccoreyes/openhie-homelab/issues)
- **Discussions**: GitHub Discussions for questions and ideas
- **Community**: Join healthcare IT and homelab communities for support

---

**Let's save lives, one commit at a time** 🏥 + 🖥️ + 🐳 = ❤️
