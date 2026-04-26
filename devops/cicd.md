# devops/cicd.md  
## CI/CD – DevOps käsiraamat

## Ülevaade

CI/CD (Continuous Integration / Continuous Delivery) on DevOpsi keskne osa.  
Eesmärk on:

- automatiseerida build → test → deploy protsess  
- vähendada vigade hulka  
- kiirendada arendust  
- tagada stabiilsed ja korduvad deployd  

Allpool on kõige olulisemad tööriistad ja mustrid.

---

# 1. CI/CD põhimõtted

## Continuous Integration (CI)

- Iga commit → automaatne build  
- Automaatne testimine  
- Koodi kvaliteedi kontroll  
- Artefaktide loomine  

## Continuous Delivery (CD)

- Automaatne deploy test/prod keskkondadesse  
- Infrastruktuur kui kood  
- GitOps / deklaratiivne deploy  

---

# 2. GitHub Actions

## Workflow fail  
`.github/workflows/build.yml`

### Näide

```yaml
name: Build and Test

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - run: npm test
```

---

# 3. GitLab CI

`.gitlab-ci.yml`

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
  artifacts:
    paths:
      - dist/

test:
  stage: test
  script:
    - npm test

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

---

# 4. Jenkins Pipeline

`Jenkinsfile`

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
    stage('Deploy') {
      steps {
        sh './deploy.sh'
      }
    }
  }
}
```

---

# 5. CI/CD mustrid

## 5.1 Build once, deploy many  
Artefakt ehitatakse üks kord → kasutatakse kõigis keskkondades.

## 5.2 Shift-left testing  
Testid jooksevad võimalikult vara.

## 5.3 Immutable deployments  
Ei muudeta jooksvaid pod’e → uus versioon = uus pod.

## 5.4 GitOps  
Deploy toimub Git’i commitide kaudu.

---

# 6. CI/CD Best Practices

- Kasuta **väikseid ja kiireid pipeline’e**  
- Kasuta **cache’i** (npm, Docker layers)  
- Ära kasuta `latest` konteineripilte  
- Kasuta **trunk-based development**  
- Tee **automaatne lint + test** igal commitil  
- Kasuta **secrets managerit** (mitte `.env` Git’is)  
- Kasuta **artefaktide versioonimist**  

---

# 7. CI/CD Anti‑Patterns

- Manuaalsed deployd  
- “Works on my machine”  
- Suured monoliitsed pipeline’id  
- Testide vahelejätmine  
- Secrets Git’is  
- Ebastabiilsed runnerid  

---

# Kokkuvõte

CI/CD muudab arenduse kiiremaks, stabiilsemaks ja automatiseeritumaks.  
GitHub Actions, GitLab CI ja Jenkins on kõige levinumad tööriistad, mis võimaldavad ehitada professionaalseid ja usaldusväärseid pipeline’e.

---