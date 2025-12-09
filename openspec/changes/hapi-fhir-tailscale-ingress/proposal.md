# Proposal: HAPI FHIR Tailscale Ingress Configuration

**Change ID**: hapi-fhir-tailscale-ingress
**Status**: Draft
**Created**: 2025-12-09

## Summary

Expose the HAPI FHIR JPA server on the `cdr.tail126a09.ts.net` subdomain via Tailscale ingress, enabling secure remote access to the clinical data repository through the Tailscale VPN network.

## Context & Motivation

Currently, HAPI FHIR is deployed in the cluster but not accessible remotely. The project already uses Tailscale for cluster access (Portainer is available via Tailscale at `portainer` hostname). This proposal extends the same pattern to HAPI FHIR, making it accessible as a clinical data repository (CDR) endpoint.

**Current State:**
- HAPI FHIR deployed via HelmRelease (commented out in kustomization)
- Uses deprecated CHGL chart
- Service type: ClusterIP, port 8080
- No external access configured

**Target State:**
- HAPI FHIR available at `cdr.tail126a09.ts.net`
- Tailscale ingress provides secure VPN-based access
- Follows the same pattern as Portainer deployment

## Related Documents

- [HAPI FHIR Official Helm Charts](https://github.com/hapifhir/hapi-fhir-jpaserver-starter/tree/master/charts/hapi-fhir-jpaserver)
- Current deployment: `clusters/pi-cluster/flux-system/openhie-apps/02-hapi-fhir-release.yaml`
- Reference implementation: `clusters/pi-cluster/flux-system/apps/portainer.yaml`

## Capabilities & Requirements

This change introduces one new capability:

1. **cdr-tailscale-access** - Expose HAPI FHIR on Tailscale subdomain
   - See: `openspec/changes/hapi-fhir-tailscale-ingress/specs/cdr-tailscale-access/spec.md`

## Design Decisions

### Architecture Pattern
Following the established pattern in the codebase (Portainer deployment), we will:
- Create a Kubernetes Ingress resource with `ingressClassName: tailscale`
- Use hostname `cdr` (Tailscale MagicDNS will resolve to `cdr.tail126a09.ts.net`)
- Maintain ClusterIP service type for the HAPI FHIR service
- TLS termination at Tailscale layer

### Prerequisites
1. HAPI FHIR HelmRelease must be enabled in kustomization
2. Tailscale operator must be deployed and operational
3. Tailscale auth credentials configured (operator-oauth secret)

## Implementation Considerations

### Scope
This change is tightly scoped to:
- Creating an Ingress resource for HAPI FHIR
- Enabling the HAPI FHIR HelmRelease in Flux
- Following existing Tailscale patterns

### Out of Scope (Future Work)
- Migrating from CHGL chart to official HAPI FHIR chart (noted as deprecated)
- Custom domain configuration beyond `cdr.tail126a09.ts.net`
- Authentication/authorization beyond Tailscale VPN access
- Load balancing or high availability considerations

## Testing Strategy

### Validation Steps
1. Deploy the ingress configuration
2. Verify `cdr.tail126a09.ts.net` resolves via Tailscale MagicDNS
3. Access FHIR API endpoint: `https://cdr.tail126a09.ts.net/fhir/metadata`
4. Verify FHIR conformance statement returns successfully
5. Confirm service remains accessible internally

### Success Criteria
- Successful access to HAPI FHIR via `cdr.tail126a09.ts.net`
- TLS/HTTPS connection established
- FHIR API responds correctly
- No disruption to internal cluster access

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Tailscale operator not configured | High | Verify operator is deployed before applying ingress |
| HAPI FHIR deployment disabled | Medium | Update kustomization to enable HelmRelease |
| Port conflicts or service misconfiguration | Medium | Test connectivity and review service ports |
| DNS resolution issues | Low | Tailscale MagicDNS typically reliable; verify Tailscale status |

## Dependencies

- Tailscale operator deployed and operational
- HAPI FHIR HelmRelease properly configured (currently commented out)
- Tailscale auth credentials (operator-oauth secret in tailscale namespace)

## Rollback Plan

If issues arise:
1. Disable the ingress resource by commenting it out
2. HAPI FHIR will remain accessible internally via ClusterIP
3. No data loss risk (stateless ingress configuration)

## Effort Estimate

- **Implementation**: ~2-3 hours
  - Create ingress manifest
  - Update kustomization to enable HAPI FHIR
  - Test and validate
- **Documentation**: 30 minutes
- **Total**: ~3 hours
