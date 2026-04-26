# devops/git.md  
## Git – DevOps käsiraamat

## Ülevaade

Git on hajus versioonihaldussüsteem, mida kasutatakse DevOpsis koodi, infrastruktuuri ja konfiguratsioonide haldamiseks.  
Allpool on kõige olulisemad käsud ja töövood.

---

# Põhikäsud

## Repo kloonimine

```bash
git clone <repo-url>
```

## Staatus

```bash
git status
```

## Failide lisamine

```bash
git add <fail>
git add .
```

## Commit

```bash
git commit -m "Sõnum"
```

## Push

```bash
git push
```

## Pull

```bash
git pull
```

---

# Harud (branches)

## Haru loomine

```bash
git branch feature-x
```

## Haru vahetamine

```bash
git checkout feature-x
```

## Haru loomine + checkout

```bash
git checkout -b feature-x
```

## Harude nimekiri

```bash
git branch
```

---

# Merge & Rebase

## Merge

```bash
git checkout main
git merge feature-x
```

## Rebase

```bash
git checkout feature-x
git rebase main
```

---

# Konfliktide lahendamine

1. Ava konfliktiga failid  
2. Paranda käsitsi  
3. Lisa failid:

```bash
git add .
```

4. Jätka:

```bash
git rebase --continue
```

---

# Logid

## Lühike logi

```bash
git log --oneline
```

## Graafiline logi

```bash
git log --oneline --graph --decorate --all
```

---

# Remote haldus

## Remote lisamine

```bash
git remote add origin <url>
```

## Remote vaatamine

```bash
git remote -v
```

---

# Tag’id

## Tag’i loomine

```bash
git tag v1.0
```

## Tag’i push

```bash
git push origin v1.0
```

---

# Git Best Practices

- Ära kasuta `git push -f` (force push) shared harudes  
- Kasuta alati **feature branch’e**  
- Tee **väikesed commitid**  
- Kasuta **selgeid commit message’e**  
- Ära kasuta `latest` tag’i infrastruktuuris  
- Kasuta `.gitignore`  

---

# .gitignore näide

```
*.log
*.tmp
node_modules/
.env
```

---

# Git töövood DevOpsis

## Feature Branch Workflow

- main → stabiilne  
- feature-x → arendus  
- PR → koodikontroll  

## GitOps Workflow

- Git = tõeallikas  
- CI → valideerib  
- CD → sünkroniseerib klastrisse  

---
Git Handbook [Git](https://git-scm.com/book/en/v2)
# Kokkuvõte

Git on DevOpsi vundament.  
Kõik infrastruktuur, kood ja konfiguratsioonid peaksid olema Git’is, et tagada jälgitavus, turvalisus ja automatiseeritus.
