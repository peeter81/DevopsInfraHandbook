# 📄 **Fail: `containers/cni.md`**

```markdown
# CNI – Container Network Interface  
## (Plugins, configuration, chaining, Kubernetes integration)

## Ülevaade
CNI (Container Network Interface) on standard, mida kasutavad:

- Kubernetes  
- containerd  
- CRI‑O  
- Podman (rootless)  

CNI määrab, kuidas konteinerid saavad IP‑aadressid, kuidas nad suhtlevad ja milliseid võrgupoliitikaid rakendatakse.

---

# 1. CNI komponendid

CNI koosneb kolmest osast:

1. **CNI plugins** – tegelikud võrgudraiverid  
2. **CNI config** – JSON/YAML konfiguratsioonifailid  
3. **CNI runtime** – containerd/CRI‑O/Kubelet, mis käivitab pluginaid  

---

# 2. CNI pluginate tüübid

## 2.1 Põhipluginad

| Plugin | Kirjeldus |
|--------|-----------|
| **bridge** | loob Linux bridge’i, NAT hosti kaudu |
| **host-local** | IPAM, jagab lokaalseid IP‑sid |
| **loopback** | vajalik igale podile |
| **macvlan** | konteiner saab MAC‑aadressi |
| **ipvlan** | sarnane macvlan’ile, kuid kiirem |
| **ptp** | point‑to‑point link |

## 2.2 Võrgustiku pluginad (overlay)

| Plugin | Kirjeldus |
|--------|-----------|
| **flannel** | lihtne overlay, VXLAN |
| **calico** | BGP routing + NetworkPolicies |
| **cilium** | eBPF, väga kiire, turvaline |
| **weave** | automaatne mesh overlay |

---

# 3. CNI kataloogid

### Pluginad:

```
/opt/cni/bin/
```

### Konfiguratsioon:

```
/etc/cni/net.d/
```

Näide failist:

```json
{
  "cniVersion": "0.4.0",
  "name": "mynet",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.22.0.0/16"
  }
}
```

---

# 4. CNI töövoog (Kubernetes)

Kui pod käivitatakse:

1. kubelet kutsub CRI runtime’i (containerd/CRI‑O)  
2. runtime kutsub CNI pluginaid  
3. CNI loob võrgu:  
   - IP aadress  
   - veth paar  
   - bridge või overlay  
4. CNI tagastab IP kubeletile  
5. Pod saab võrguühenduse  

---

# 5. CNI chaining (mitu pluginat järjest)

Näide:

```json
{
  "name": "mynet",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "cni0"
    },
    {
      "type": "portmap",
      "capabilities": {"portMappings": true}
    }
  ]
}
```

---

# 6. Kubernetes CNI valik

### 6.1 Flannel
- lihtne  
- VXLAN  
- ei toeta NetworkPolicies  

### 6.2 Calico
- BGP routing  
- NetworkPolicies  
- tootmisklassika  

### 6.3 Cilium
- eBPF  
- väga kiire  
- turvaline  
- service mesh ilma sidecar’ita  

### 6.4 Weave
- automaatne mesh  
- lihtne seadistada  

---

# 7. Troubleshooting

### 7.1 Pod stuck in `ContainerCreating`

Kontrolli CNI kataloogi:

```bash
ls /etc/cni/net.d/
```

### 7.2 CNI plugin missing

```bash
ls /opt/cni/bin/
```

### 7.3 Kubelet logid

```bash
journalctl -u kubelet
```

### 7.4 IP konfliktid

Kontrolli subnetti:

```bash
cat /etc/cni/net.d/*.conf
```

---

# 8. Best Practices

- Kasuta **Calico** või **Cilium** tootmises  
- Ära kasuta Flannelit, kui vajad NetworkPolicies  
- Kontrolli alati CNI konfiguratsiooni pärast cluster reset’i  
- Kasuta eBPF (Cilium) suure jõudluse jaoks  
- Ära muuda CNI config’e jooksva klastri peal  

---

# Kokkuvõte
CNI on konteinerivõrkude standard, mis määrab, kuidas podid saavad IP‑sid ja suhtlevad.  
Kubernetes kasutab CNI pluginaid nagu Calico, Cilium ja Flannel, et pakkuda skaleeritavat ja turvalist võrgukihti.
```
