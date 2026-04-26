# 📄 **Fail: `automation/ansible.md`**

```markdown
# Ansible – Automation käsiraamat  
## (Inventory, Playbooks, Modules, Variables, Roles, Handlers, Vault)

## Ülevaade
Ansible on agentless automatiseerimisplatvorm, mis kasutab SSH-d ja YAML‑i.  
Sobib:

- serverite seadistamiseks  
- tarkvara paigaldamiseks  
- konfiguratsioonihalduseks  
- orkestreerimiseks  
- DevOps töövoogudeks  

---

# 1. Ansible arhitektuur

Komponendid:

- **Control Node** – kus Ansible töötab  
- **Managed Nodes** – serverid, kuhu ühendutakse  
- **Inventory** – serverite nimekiri  
- **Playbooks** – automatiseerimise loogika  
- **Modules** – tegevused (package, file, service jne)  
- **Roles** – korduvkasutatavad struktuurid  

---

# 2. Inventory

## 2.1 Lihtne inventory

```
[web]
192.168.1.10
192.168.1.11

[db]
db1 ansible_host=192.168.1.20
```

## 2.2 Gruppide sees grupid

```
[frontend]
web1
web2

[backend]
api1
api2

[app:children]
frontend
backend
```

---

# 3. Ad-hoc käsud

Ping:

```bash
ansible all -m ping
```

Paketi install:

```bash
ansible web -m apt -a "name=nginx state=present" -b
```

Faili kopeerimine:

```bash
ansible all -m copy -a "src=app.conf dest=/etc/app.conf"
```

---

# 4. Playbooks

Playbook on YAML-fail, mis kirjeldab tegevusi.

Näide:

```yaml
- hosts: web
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

Käivita:

```bash
ansible-playbook site.yaml
```

---

# 5. Variables

## 5.1 Playbooki sees

```yaml
vars:
  port: 8080
```

## 5.2 Host vars

`host_vars/web1.yml`

```yaml
env: production
```

## 5.3 Group vars

`group_vars/web.yml`

```yaml
nginx_port: 80
```

---

# 6. Templates (Jinja2)

`nginx.conf.j2`:

```
server {
  listen {{ nginx_port }};
}
```

Playbook:

```yaml
- template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

---

# 7. Handlers

Käivitatakse ainult siis, kui midagi muutus.

```yaml
tasks:
  - name: Update config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

---

# 8. Roles

Rolli struktuur:

```
roles/
  web/
    tasks/
    handlers/
    templates/
    files/
    vars/
    defaults/
```

Kasutamine:

```yaml
- hosts: web
  roles:
    - web
```

---

# 9. Ansible Vault

Krüpteeri tundlikud failid:

```bash
ansible-vault encrypt secrets.yml
```

Ava:

```bash
ansible-vault decrypt secrets.yml
```

Käivita playbook vaultiga:

```bash
ansible-playbook site.yml --ask-vault-pass
```

---

# 10. Ansible Galaxy

Installi valmis rolle:

```bash
ansible-galaxy install geerlingguy.nginx
```

---

# 11. Best Practices

- Kasuta **roles** projekti struktuuri jaoks  
- Kasuta **group_vars** ja **host_vars**  
- Kasuta **Vault** paroole ja võtmeid hoidmiseks  
- Ära pane paroole Git’i  
- Kasuta **handlers** teenuste restartimiseks  
- Hoia playbookid idempotentsed  
- Kasuta **ansible-lint** kvaliteedikontrolliks  

---

# Kokkuvõte
Ansible on võimas, agentless automatiseerimisvahend, mis sobib serverite haldamiseks, konfiguratsiooniks ja orkestreerimiseks.  
Inventory + Playbooks + Roles = skaleeritav ja korduvkasutatav automatiseerimine.
```
