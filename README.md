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

``text
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
                                
## Prérequis

- Linux (Ubuntu 22.04/24.04 recommandé)
- Docker
- kind v0.22.0+
- kubectl & helm v3

```bash
sudo apt update && sudo apt install docker.io -y

# Installation de kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/

