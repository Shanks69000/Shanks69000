# Flux réseau — Portfolio DevOps
 
## infra-as-code-homelab
 
```
Terraform (poste local)  --HTTPS/8006-->  API Proxmox VE
Ansible controller       --SSH/22-->      bastion, k8s-master, k8s-worker1/2, db-server
k8s-workers               --TCP/6443-->    k8s-master (API server K3s)
```
 
## docker-compose-stack
 
```
Client            --HTTPS/443-->  nginx-rp
nginx-rp          --auth_request--> authelia:9091
authelia          --TCP/6379-->   redis
nginx-rp          --HTTP interne--> homer:8080
nginx-rp          --HTTP interne--> uptime-kuma:3001
```
 
## ci-cd-pipeline-demo
 
```
GitHub Actions runner  --HTTPS-->  GHCR (push image)
Client                 --TCP/5000--> Flask (/, /health)
```
 
## infra-health-checker
 
```
CronJob (K8s ou GitHub Actions)  --HTTPS/443-->  github.com, hub.docker.com, registry.terraform.io
CronJob                          --TCP/22-->     github.com (SSH check)
CronJob                          --TCP/53-->     8.8.8.8 (DNS check)
CronJob                          --TLS/443-->    github.com, hub.docker.com (cert check)
GitHub Actions                   --HTTPS-->      GHCR (push image)
```
 
## wiki-sync-bot
 
```
Bot Python  --HTTPS/443 (API REST)-->  instance Outline (upsert documents)
Bot Python  --local FS-->              output/*.md (mode dry-run)
```
 
## k8s-apps-deployment
 
```
Client                  --HTTP/HTTPS-->  Traefik Ingress
Traefik                 --HTTP interne--> homer / it-tools (namespace apps)
Pods tier:frontend      --UDP/53-->      CoreDNS
Pods tier:frontend      --TCP/8080,80--> Ingress controller
ArgoCD controller       --HTTPS (poll)--> GitHub repo (base/)
ArgoCD                  --K8s API interne--> API server K3s
```
 
## helm-charts
 
```
helm install  --K8s API-->   CronJob infra-health-checker (namespace default)
kubelet       --HTTPS-->     GHCR (image pull)
```
 
## ansible-drift-detector
 
```
Runner self-hosted (WAN dynamique)
    --UDP/41820-->  Bbox (NAT)
    --DNAT-->       machine hôte (192.168.1.63)
    --UDP/51820-->  bastion (WireGuard, 10.10.1.11)
 
Cron local (duck.sh)  --HTTPS-->  DuckDNS API
 
Runner (tunnel, 10.200.200.2)
    --SSH/22 via 10.200.200.0/24-->  bastion / k8s-master / workers / db-server (10.10.1.0/24)
 
GitHub Actions job  --SMTPS/465-->  smtp.gmail.com --> NOTIFY_EMAIL
 
Runner  --HTTPS-->  GitHub repo (commit reports/)
```
 
---
 
Réf architecture tunnel détaillée : `docs/wireguard-tunnel-setup.md` (repo `ansible-drift-detector`).