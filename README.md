# Déploiement Free5GC & UERANSIM sur Kubernetes (Kind)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Kind-blue?logo=kubernetes)](https://kind.sigs.k8s.io/)
[![Free5GC](https://img.shields.io/badge/Free5GC-v3.3.0-brightgreen)](https://free5gc.org)
[![UERANSIM](https://img.shields.io/badge/UERANSIM-v3.2.6-orange)](https://github.com/aligungr/UERANSIM)

**Projet réalisé par IHADDADEN SOHEIB – 2025**

Simulation complète d’un réseau **5G Standalone (SA)** local avec le cœur open-source **Free5GC** et la RAN simulée **UERANSIM**, le tout déployé sur un cluster **Kubernetes Kind** (Docker).

## Objectifs
- Créer un cluster Kubernetes local avec Kind  
- Déployer Free5GC v3.3.0 (AMF, SMF, UPF, NRF, UDM, AUSF, PCF) via Helm  
- Déployer UERANSIM v3.2.6 (gNB + UE)  
- Enregistrer un UE dans le réseau 5G  
- Établir une PDU Session  
- Observer les échanges NAS, NGAP et PFCP en temps réel  

## Architecture

```text
+-----------------+      N1/N2      +--------------------+
|     UE (SIM)    | <-------------> |        AMF         |
+-----------------+                 +--------------------+
           |                                |
           |  RRC / NGAP                    |
           v                               
+-----------------+      N3        +--------------------+
|   gNB (UERANSIM)| <------------> |        UPF         |
+-----------------+                +--------------------+
                                         |
                                         | N6
                                         v
                                (Internet via NAT)

```
## Prérequis

- Linux (Ubuntu 22.04/24.04 recommandé)
- Docker
- kind v0.22.0+
- kubectl & helm v3

```bash
sudo apt update && sudo apt install docker.io -y
```
# Installation de kind
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/

```


### Création du cluster Kind
```bash
kind create cluster --name free5gc
```

### Déploiement de Free5GC
```bash
helm repo add free5gc https://free5gc.org/charts
helm repo update
helm install free5gc-premier free5gc/free5gc -n free5gc --create-namespace
```

### Déploiement UERANSIM (gNB + UE)
```bash
helm install ueransim-premier free5gc/ueransim -n free5gc
```

### Configuration du gNB (obligatoire)
```bash
kubectl get configmap gnb-configmap -n free5gc -o yaml > /tmp/gnb.yaml
# Modifier ngapIp (AMF) et gtpIp (gNB) selon votre cluster
kubectl apply -f /tmp/gnb.yaml
kubectl delete pod -n free5gc -l component=gnb
```
## Création de l’abonné (WebUI)

URL : http://localhost:3000 → *admin / free5gc*

| Champ | Valeur |
|-------|--------|
| IMSI  | 208930000000003 |
| Key   | 8baf473f2f8fd09487ccb... |
| OPC   | 8e27b6af0e692e750f326... |
| DNN   | internet |
| SST   | 1 |
| SD    | 010203 |

---

## Tests réalisés

### ✔️ Registration UE
```bash
kubectl logs -n free5gc <UE_POD> | grep -i Registration
# → Registration Accept
```

### ✔️ Connexion gNB → AMF
```bash
kubectl logs -n free5gc <GNB_POD> | grep NGSetup
# → NG Setup successful
```

### ✔️ Association PFCP (SMF ↔ UPF)
```bash
kubectl logs -n free5gc -l app=smf | grep PFCP
kubectl logs -n free5gc -l app=upf | grep PFCP
# → PFCP Association Setup Accepted
```

### ✔️ Établissement de PDU Session
```bash
kubectl logs -n free5gc <UE_POD> | grep "PDU Session Establishment Accept"
```
# Déploiement Free5GC + UERANSIM sur Kubernetes (Kind)

## Vérification du plan utilisateur (GTP-U)

```bash
kubectl logs -n free5gc -l app=upf | grep GTP
# → [INFO][UPF][Gtp5g] Forwarder started
```

---

## Ping UE → UPF (N6)

```bash
kubectl exec -it -n free5gc <UE_POD> -- ping -I uesimtun0 10.10.6.20
# → 0% packet loss
```

---

## IP affectée à l’UE

```bash
kubectl exec -it -n free5gc <UE_POD> -- ip addr show uesimtun0
# → inet 10.1.0.X/32
```

---

## Gateway N6 & NAT sur la VM hôte (accès Internet)

```bash
# Ajouter la gateway
sudo ip addr add 10.10.6.1/24 dev br-$(docker network ls --filter name=kind -q | head -n1 | cut -c1-12)
sudo ip link set br-$(docker network ls --filter name=kind -q | head -n1 | cut -c1-12) up

# Activer le forwarding et le NAT
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -t nat -A POSTROUTING -s 10.10.6.0/24 -o enp0s3 -j MASQUERADE
```

---

## Test final : UE → Internet

```bash
kubectl exec -it -n free5gc <UE_POD> -- ping -I uesimtun0 8.8.8.8
# → Réponses ICMP
```

---

## Vérification finale du pipeline complet

| Étape            | Vérification                | Statut |
|------------------|-----------------------------|--------|
| RAN → AMF        | NG Setup OK                 | Done   |
| UE → AMF         | Registration Accept         | Done   |
| SMF → UPF        | PFCP Association OK         | Done   |
| UE → SMF/UPF     | PDU Session Accept          | Done   |
| UE → UPF (N6)    | Ping 10.10.6.20 OK          | Done   |
| UE → Internet    | Ping 8.8.8.8 OK (via NAT)   | Done   |

---

## Nettoyage

```bash
kind delete cluster --name free5gc
```

---

## License

Ce projet est sous licence **MIT** – voir LICENSE.





