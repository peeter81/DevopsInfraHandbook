# devops/gitops.md  
## GitOps – DevOps käsiraamat

## Ülevaade

GitOps on lähenemine, kus **Git on kogu infrastruktuuri ja rakenduste tõeallikas**.  
Kõik muudatused tehakse Git’i commitide kaudu ja klaster sünkroniseerib end automaatselt Git’i seisuga.

GitOps =  
**Declarative + Versioned + Pull‑based + Continuous Reconciliation**

Peamised tööriistad:

- **ArgoCD**
- **FluxCD**

---

# 1. GitOps põhimõtted

## Declarative  
Kõik konfiguratsioonid (YAML, Helm, Kustomize) on Git’is.

## Versioned  
Git ajalugu = audit log.

## Pull‑based  
Klastri sees olev agent tõmbab muudatused Git’ist.

## Continuous Reconciliation  
Agent võrdleb:

- desired state (Git)  
- actual state (cluster)

ja parandab drift’i.

---

# 2. ArgoCD

## Olulised käsud

```
argocd login <server>
argocd app create <app>
argocd app sync <app>
argocd app diff <app>
argocd app rollback <app> <revision>
```

## ArgoCD Application näide

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web
spec:
  project: default
  source:
    repoURL: https://github.com/example/web
    path: overlays/prod
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Selgitus

- **automated** → automaatne deploy  
- **prune** → kustutab Git’ist eemaldatud ressursid  
- **selfHeal** → parandab drift’i  

---

# 3. FluxCD

## Bootstrap

```
flux bootstrap github \
  --owner=peeter \
  --repository=infra \
  --branch=main \
  --path=clusters/prod
```

## Sünkroniseerimine

```
flux reconcile kustomization app
```

## Ressursside vaatamine

```
flux get kustomizations
flux get sources git
```

---

# 4. GitOps kaustastruktuur

```
infra/
│
├── clusters/
│   ├── dev/
│   ├── stage/
│   └── prod/
│
└── apps/
    ├── web/
    │   ├── base/
    │   └── overlays/
    └── backend/
        ├── base/
        └── overlays/
```

---

# 5. GitOps töövoog

```
Developer Commit
       |
       v
+----------------------+
|      Git Repo        |
+----------------------+
       |
       v
+----------------------+
|   GitOps Controller  |
| (ArgoCD / FluxCD)    |
+----------------------+
       |
       v
+----------------------+
| Kubernetes Cluster   |
+----------------------+
```

---

# 6. GitOps Best Practices

- **Ära tee kubectl apply otse prod’i**  
- Kasuta **Kustomize** või **Helm** baase + overlay’sid  
- Kasuta **automated sync** + **selfHeal**  
- Kasuta **sealed-secrets** või **external-secrets**  
- Tee **väikesed commitid** ja **PR review**  
- Kasuta **branch protection rules**  

---

# 7. GitOps Anti‑Patterns

- Manuaalsed muudatused klastris  
- “latest” konteineripildid  
- Secrets Git’is plaintextina  
- Kustomize + Helm segamine valesti  
- Ignore‑rules liiga laiad (drift jääb märkamata)  

---

# Kokkuvõte

GitOps muudab infrastruktuuri haldamise turvaliseks, jälgitavaks ja automatiseerituks.  
ArgoCD ja FluxCD on tööstusstandardid, mis tagavad stabiilsed ja ennustatavad deployd.

---

## Kui soovid, võin:

- puhastada ülejäänud failid  
- kirjutada automaatse skripti, mis tuvastab ja eemaldab valed Unicode märgid  
- anda juhised, kuidas vältida encoding‑probleeme edaspidi  
