# Capability: CDR Tailscale Access

**ID**: cdr-tailscale-access
**Change**: hapi-fhir-tailscale-ingress

## ADDED Requirements

### Requirement: HAPI FHIR accessible on Tailscale subdomain
The HAPI FHIR server SHALL be accessible at the designated Tailscale subdomain for secure remote access to the clinical data repository.

#### Scenario: Expose HAPI FHIR on Tailscale subdomain
When accessing the HAPI FHIR server remotely
Then it should be available at `cdr.tail126a09.ts.net`
And it should route through the Tailscale VPN
And it should use HTTPS/TLS for secure communication

### Requirement: Tailscale ingress configuration
The system SHALL configure a Kubernetes ingress resource with Tailscale-specific settings to route traffic to the HAPI FHIR service using MagicDNS resolution.

#### Scenario: Configure Tailscale ingress for HAPI FHIR
Given HAPI FHIR is deployed via Flux HelmRelease
When the ingress is configured with `ingressClassName: tailscale`
And the host is set to `cdr`
Then the service should be reachable via Tailscale MagicDNS

### Requirement: Internal service accessibility
The HAPI FHIR service SHALL remain accessible within the cluster for internal services and SHALL NOT rely on ingress for local communication.

#### Scenario: HAPI FHIR service remains accessible internally
Given Tailscale ingress is configured
Then the service should remain accessible within the cluster at `hapi-fhir-jpaserver.default.svc.cluster.local:8080`
And it should continue to use ClusterIP service type

### Requirement: TLS termination at Tailscale
The system SHALL implement TLS termination at the Tailscale ingress layer to provide secure external access while maintaining efficient internal cluster communication.

#### Scenario: TLS termination at Tailscale
When accessing HAPI FHIR via the Tailscale domain
Then TLS should be terminated at the Tailscale ingress layer
And the connection from Tailscale to the pod should be over the cluster network
