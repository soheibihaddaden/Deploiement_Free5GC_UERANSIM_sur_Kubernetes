# Déploiement Free5GC + UERANSIM sur Kubernetes (Kind)

---

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

---
