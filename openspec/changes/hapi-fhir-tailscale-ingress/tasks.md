# Tasks: HAPI FHIR Tailscale Ingress

**Change ID**: hapi-fhir-tailscale-ingress
**Status**: Pending

## Implementation Tasks

### Phase 1: Enable HAPI FHIR Deployment

1. **Enable HAPI FHIR HelmRelease in Flux**
   - [ ] Uncomment `openhie-apps/01-helm-repositories.yaml` in kustomization
   - [ ] Uncomment `openhie-apps/02-hapi-fhir-release.yaml` in kustomization
   - [ ] Commit changes to repository
   - [ ] Wait for Flux to sync (or force sync)
   - [ ] Verify HAPI FHIR deployment status
   - **Validation**: `kubectl get helmrelease -n flux-system hapi-fhir-jpaserver` shows deployed

### Phase 2: Create Tailscale Ingress

2. **Create HAPI FHIR Service**
   - [ ] Create `clusters/pi-cluster/flux-system/openhie-apps/02-hapi-fhir-service.yaml`
   - [ ] Define ClusterIP service for HAPI FHIR (port 8080)
   - [ ] Reference the pods created by HelmRelease
   - [ ] Commit the service manifest

3. **Create Tailscale Ingress Resource**
   - [ ] Create `clusters/pi-cluster/flux-system/openhie-apps/03-hapi-fhir-ingress.yaml`
   - [ ] Set `ingressClassName: tailscale`
   - [ ] Configure host as `cdr`
   - [ ] Route traffic to HAPI FHIR service on port 8080
   - [ ] Ensure path: `/` with pathType: Prefix
   - [ ] Commit the ingress manifest

4. **Update Kustomization**
   - [ ] Add service and ingress resources to `openhie-apps/kustomization.yaml`
   - [ ] Verify all resources properly ordered
   - [ ] Commit kustomization changes

### Phase 3: Testing and Validation

5. **Verify Deployment**
   - [ ] Check HAPI FHIR pod status: `kubectl get pods -n default -l app.kubernetes.io/name=hapi-fhir-jpaserver`
   - [ ] Verify service created: `kubectl get service -n default hapi-fhir-jpaserver`
   - [ ] Confirm ingress created: `kubectl get ingress -n default hapi-fhir-ingress`
   - [ ] Check Tailscale machine appears in admin panel

6. **Validate External Access**
   - [ ] Ensure Tailscale connected on local machine
   - [ ] Test DNS resolution: `nslookup cdr.tail126a09.ts.net`
   - [ ] Access FHIR metadata endpoint: `curl https://cdr.tail126a09.ts.net/fhir/metadata`
   - [ ] Verify FHIR conformance statement returns correctly
   - [ ] Test FHIR API functionality with sample requests

7. **Validate Internal Access**
   - [ ] Access service via cluster DNS: `curl http://hapi-fhir-jpaserver.default.svc.cluster.local:8080/fhir/metadata`
   - [ ] Verify internal connectivity still works
   - [ ] Confirm no disruption to internal services

### Phase 4: Documentation

8. **Update Project Documentation**
   - [ ] Document the `cdr.tail126a09.ts.net` endpoint in README
   - [ ] Add usage examples for accessing FHIR API
   - [ ] Document any authentication requirements
   - [ ] Note the deprecation notice for CHGL chart (future migration)

## Work Items Dependencies

- **Tailscale operator must be deployed** (currently in `apps/tailscale-operator.yaml`)
- **Tailscale auth configured** (operator-oauth secret in tailscale namespace)
- **HAPI FHIR HelmRelease properly configured** (currently commented out)

## Parallelizable Work

- Task 1 (enable HelmRelease) can be done independently
- Tasks 2-4 (create manifests) can be done in parallel with testing preparations
- Task 8 (documentation) can be done in parallel with other work

## Rollback Steps

If validation fails:
1. Comment out ingress resource in kustomization
2. Delete ingress: `kubectl delete ingress hapi-fhir-ingress -n default`
3. HAPI FHIR remains available internally via ClusterIP
4. Debug issues without affecting internal access

## Success Criteria

- [ ] `cdr.tail126a09.ts.net` resolves and is accessible
- [ ] HTTPS/TLS connection successfully established
- [ ] FHIR metadata endpoint returns conformance statement
- [ ] FHIR API responds correctly to queries
- [ ] Internal cluster access remains functional
- [ ] No errors in pod logs related to ingress/networking
