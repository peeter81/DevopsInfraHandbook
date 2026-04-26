DevOps-Infra-Observability Handbook



„DevOps \& Infrastructure Engineering Handbook“  

mille sees on kolm suurt sammast:



DevOps  

Git, CI/CD, Kubernetes, Kustomize, GitOps, Service Mesh, Container Orchestration, Cloud-init, AWS/Azure/GCP, automatsioon, pipelines.



Infrastructure / Linux Internals / Networking  

Kernel internals, hugepages, netfilter, conntrack, VXLAN, OVS, SR-IOV, DPDK, storage, security (SELinux), systemd templates, virtualization.



Observability  

Prometheus, Grafana, Loki, Promtail, Elasticsearch, Jaeger, Tempo, OpenTelemetry, Alertmanager Routing \& Silences, RabbitMQ observability, SLO/SLI, distributed tracing patterns, synthetic monitoring jne





See on täpselt see, mida üks päris ettevõtte sisemine „Platform Engineering Handbook“ sisaldaks.



NB! Osad peatükid võivad puududa (vajavad lisamist)!





handbook/
│
├── README.md
│
├── devops/
│   ├── git.md
│   ├── cicd.md
│   ├── kubernetes.md
│   ├── kustomize.md
│   ├── gitops.md
│   └── service-mesh.md
│
├── networking/
│   ├── netfilter.md
│   ├── netfilter-hooks.md
│   ├── nftables.md
│   ├── conntrack.md
│   ├── vxlan.md
│   ├── ovs.md
│   ├── dpdk.md
│   ├── sriov.md
│   ├── bonding.md
│   ├── vrf.md
│   ├── mpls.md
│   └── xdp.md
│
├── linux-internals/
│   ├── scheduling.md
│   ├── memory.md
│   ├── hugepages.md
│   ├── processes.md
│   ├── filesystems.md
│   ├── networking-stack.md
│   ├── storage.md
│   ├── security.md
│   ├── boot.md
│   ├── logging.md
│   ├── kernel.md
│   ├── namespaces.md
│   └── cgroups.md
│
├── storage/
│   ├── lvm-thin.md
│   ├── lvm-cache.md
│   ├── lvm-raid.md
│   ├── zfs.md
│   ├── lvm-snapshots.md
│   ├── lvm-metadata.md
│   └── device-mapper.md
│
├── security/
│   ├── selinux.md
│   ├── fips.md
│   ├── ssh-hardening.md
│   ├── firewall.md
│   ├── tpm2.md
│   ├── secure-boot.md
│   └── luks-tpm.md
│
├── containers/
│   ├── docker.md
│   ├── podman.md
│   ├── kubernetes.md
│   ├── helm.md
│   ├── containerd.md
│   ├── crio.md
│   ├── kind.md
│   ├── minikube.md
│   ├── openshift.md
│   ├── openshift-operators.md
│   ├── docker-swarm.md
│   ├── container-security.md
│   ├── container-networking.md
│   ├── cni.md
│   ├── service-mesh.md
│   ├── container-storage.md
│   ├── container-logging.md
│   ├── container-monitoring.md
│   ├── container-orchestration.md
│   ├── kubernetes-basics.md
│   ├── kubernetes-advanced.md
│   ├── kubernetes-networking.md
│   ├── kubernetes-storage.md
│   ├── kubernetes-security.md
│   ├── kubernetes-observability.md
│   ├── kubernetes-scheduling.md
│   ├── kubernetes-ingress.md
│   ├── kubernetes-jobs.md
│   ├── kubernetes-statefulsets.md
│   ├── kubernetes-daemonsets.md
│   ├── kubernetes-configmaps-secrets.md
│   ├── kubernetes-operators.md
│   ├── kubernetes-helm.md
│   └── image-optimization.md
│
├── automation/
│   ├── ansible.md
│   ├── ci-cd.md
│   ├── packer.md
│   ├── cloud-init.md
│   ├── cloud/aws.md
│   ├── azure.md
│   ├── gcp.md
│   ├── docker.md
│   ├── systemd-templates.md
│   └── terraform.md
│
└── observability/
    ├── prometheus.md
    ├── grafana.md
    ├── loki.md
    ├── loki-promtail.md
    ├── elastic.md
    ├── opentelemetry.md
    ├── tempo.md
    ├── rabbitmq.md
    ├── alertmanager.md
    └── jaeger-otel.md

