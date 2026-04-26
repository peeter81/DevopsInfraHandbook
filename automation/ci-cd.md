# 📄 **Fail: `automation/ci-cd.md`**

```markdown
# CI/CD – Automation käsiraamat  
## (Continuous Integration, Continuous Delivery, Pipelines, GitHub Actions, GitLab CI, Jenkins, ArgoCD)

## Ülevaade
CI/CD (Continuous Integration / Continuous Delivery) on DevOps‑i keskne osa.  
Eesmärk:

- automatiseeritud build  
- automatiseeritud testid  
- automatiseeritud deploy  
- kiire ja turvaline tarkvaratarne  

CI/CD koosneb kahest etapist:

- **CI** – koodi integreerimine, build, testid  
- **CD** – automaatne deploy (või manuaalne kinnitusega)

---

# 1. Continuous Integration (CI)

CI käivitub iga commit’i või pull request’i peale.

Tüüpilised sammud:

1. Checkout kood  
2. Install sõltuvused  
3. Build  
4. Testid  
5. Artefaktide loomine  
6. Raportid  

---

# 2. Continuous Delivery / Deployment (CD)

CD automatiseerib rakenduse tarnimise:

- staging  
- production  
- blue/green deploy  
- canary deploy  
- rollback  

Tööriistad:

- ArgoCD  
- FluxCD  
- Spinnaker  
- GitHub Actions  
- GitLab CI  
- Jenkins  

---

# 3. GitHub Actions

Näide `.github/workflows/build.yml`:

```yaml
name: Build and Test

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install deps
        run: npm install
      - name: Run tests
        run: npm test
```

---

# 4. GitLab CI

`.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - npm install
    - npm run build

test:
  stage: test
  script:
    - npm test

deploy:
  stage: deploy
  script:
    - ./deploy.sh
```

---

# 5. Jenkins

Jenkinsfile:

```groovy
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'npm install'
      }
    }
    stage('Test') {
      steps {
        sh 'npm test'
      }
    }
  }
}
```

---

# 6. ArgoCD (GitOps)

ArgoCD jälgib Git repo’t ja sünkroniseerib Kubernetes klastri Gitis oleva soovitud seisuga.

GitOps põhimõte:

```
Git = tõeallikas
ArgoCD = sünkroniseerija
Kubernetes = sihtkeskkond
```

ArgoCD eelised:

- automaatne deploy  
- automaatne rollback  
- audit logid  
- visuaalne UI  
- multi‑cluster tugi  

---

# 7. CI/CD pipeline’i tüüpiline struktuur

```
CI:
  - lint
  - build
  - unit tests
  - integration tests
  - security scan
  - artifact upload

CD:
  - deploy to staging
  - smoke tests
  - manual approval (optional)
  - deploy to production
  - post-deploy tests
```

---

# 8. Artefaktid

CI/CD pipeline toodab artefakte:

- Docker image  
- binaarid  
- Helm chart  
- Packer image  
- logid  
- testiraportid  

---

# 9. Secrets CI/CD-s

Ära hoia paroole Git’is.

Kasuta:

- GitHub Secrets  
- GitLab Variables  
- Jenkins Credentials  
- HashiCorp Vault  
- Azure Key Vault  
- AWS Secrets Manager  

---

# 10. Best Practices

- Kasuta **GitOps** Kubernetes deploy’de jaoks  
- Kasuta **lint + test + build** CI-s  
- Kasuta **automaatset rollback’i**  
- Ära hoia paroole repo’s  
- Kasuta **branch protection rules**  
- Kasuta **code review** enne merge’i  
- Kasuta **canary deploy** tootmises  
- Kasuta **ArgoCD** või **FluxCD** Kubernetes keskkondades  

---

# Kokkuvõte
CI/CD automatiseerib kogu tarkvaratarne protsessi — alates build’ist kuni tootmisesse jõudmiseni.  
GitHub Actions, GitLab CI, Jenkins ja ArgoCD on kõige levinumad tööriistad.
```
