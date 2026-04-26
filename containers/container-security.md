# 📄 **Fail: `containers/container-security.md`**

```markdown
# Container Security – Containers käsiraamat  
## (Hardening, scanning, runtime security, policies, supply chain)

## Ülevaade
Konteinerite turvalisus hõlmab:

- image turvalisust  
- runtime turvalisust  
- võrguturvet  
- õiguste haldust  
- supply chain kaitset  
- Kubernetes poliitikaid  

See fail koondab parimad praktikad tootmiskeskkondade jaoks.

---

# 1. Image Security

## 1.1 Kasuta väikseid base image’eid
- alpine  
- distroless  
- scratch  

## 1.2 Ära hoia image’is:
- paroole  
- SSH võtmeid  
- .env faile  
- build tööriistu  

## 1.3 Kasuta `.dockerignore`
Välista:
```
.git
node_modules
*.log
.env
```

## 1.4 Allkirjasta image’id
Kasuta:
- cosign  
- Notary v2  

Näide:

```bash
cosign sign myapp:1.0
```

---

# 2. Vulnerability Scanning

Kasuta skannereid:

### Trivy
```bash
trivy image myapp:latest
```

### Grype
```bash
grype myapp:latest
```

### Docker Scout
```bash
docker scout cves myapp
```

Skanni:
- base image  
- sõltuvused  
- OS paketid  

---

# 3. Least Privilege

## 3.1 Ära jookse root kasutajana

Dockerfile:

```dockerfile
USER appuser
```

## 3.2 Piira õigusi

```yaml
securityContext:
  runAsUser: 1000
  runAsNonRoot: true
  readOnlyRootFilesystem: true
```

---

# 4. Runtime Security

## 4.1 Seccomp

```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
```

## 4.2 AppArmor

```yaml
annotations:
  container.apparmor.security.beta.kubernetes.io/web: runtime/default
```

## 4.3 Capabilities

Eemalda kõik:

```yaml
securityContext:
  capabilities:
    drop: ["ALL"]
```

Lisa ainult vajalikud:

```yaml
add: ["NET_BIND_SERVICE"]
```

---

# 5. Network Security

## 5.1 NetworkPolicies

Näide:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

## 5.2 Zero Trust võrgumudel

- lubatud ainult vajalik liiklus  
- kõik muu keelatud  

---

# 6. Secrets Management

## 6.1 Ära hoia paroole image’is

Kasuta:
- Kubernetes Secrets  
- HashiCorp Vault  
- SOPS  
- Sealed Secrets  

## 6.2 Secret mount

```yaml
env:
  - name: DB_PASS
    valueFrom:
      secretKeyRef:
        name: db
        key: password
```

---

# 7. Supply Chain Security

## 7.1 SBOM (Software Bill of Materials)

Loo SBOM:

```bash
syft myapp:latest
```

## 7.2 Allkirjasta SBOM

```bash
cosign attest --predicate sbom.json --type spdx myapp:latest
```

## 7.3 Kontrolli image päritolu

- kasuta privaatseid registriesid  
- kasuta image pull poliitikaid  

---

# 8. Kubernetes Policy Enforcement

## 8.1 OPA Gatekeeper

Näide poliitika:

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPSPNoPrivileged
metadata:
  name: no-privileged
```

## 8.2 Kyverno

Näide:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-non-root
spec:
  validationFailureAction: enforce
  rules:
    - name: check-root
      match:
        resources:
          kinds:
            - Pod
      validate:
        message: "Root user not allowed"
        pattern:
          spec:
            containers:
              - securityContext:
                  runAsNonRoot: true
```

---

# 9. Logging & Monitoring

Kasuta:
- Falco (runtime threat detection)  
- Sysdig Secure  
- Aqua Security  
- Datadog Runtime Security  

Falco näide:

```bash
falco -r rules.yaml
```

---

# 10. Best Practices

- Ära jookse root kasutajana  
- Kasuta read-only filesystem’i  
- Kasuta seccomp + AppArmor profiile  
- Kasuta NetworkPolicies  
- Skanni image’eid regulaarselt  
- Kasuta SBOM‑e ja allkirjastamist  
- Kasuta OPA/Kyverno poliitikaid  
- Ära hoia paroole image’is  
- Kasuta privaatseid registriesid  

---

# Kokkuvõte
Container security on mitmekihiline: image turvalisus, runtime piirangud, võrgupoliitikad, secrets haldus ja supply chain kaitse.  
Tugev turvamudel nõuab kõigi nende kihtide koos kasutamist.
```
