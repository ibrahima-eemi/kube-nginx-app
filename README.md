# kube-nginx-app — Apache Reverse Proxy + Kubernetes

Déploiement d'une application statique dans un cluster Kubernetes avec un reverse proxy Apache pour exposer l'application localement et sur le réseau local **sans utiliser MetalLB**.

---

## Prérequis

- Kubernetes (ex. : cluster k3s, kubeadm, ou kind)
- Apache2 installé sur l'hôte
- Modules Apache activés : `proxy`, `proxy_http`
- Port `8080` exposé en NodePort (configurable)

---

## Contenu du projet

```bash
kube-nginx-app/
├── configmap.yaml        # Contient le HTML à déployer
├── deployment.yaml       # Déploiement Apache + Probes
├── service.yaml          # Service NodePort (expose sur 30080)
├── ingress.yaml          # (optionnel) Ingress NGINX
├── index.html            # Fichier HTML "Hello Kubernetes !"
└── README.md             # Ce fichier
```

## Déploiement Kubernetes

### Appliquer les fichiers YAML

```bash
kubectl apply -f .
```

### Vérifier les pods

```bash
kubectl get pods
```

### Vérifier le service

```bash
kubectl get svc apache-service
```
Vérifie que le NodePort est exposé (par défaut : 30080)

### Vérifier les endpoints

```bash
kubectl get endpoints apache-service
```

## Configuration Apache (Reverse Proxy)

### /etc/apache2/sites-available/kube-proxy.conf

```apache
<VirtualHost *:80>
    ServerName apache.local
    ServerAlias 10.0.10.xxx

    ProxyPreserveHost On
    ProxyPass / http://localhost:8080/
    ProxyPassReverse / http://localhost:8080/
</VirtualHost>
```

### Activer le reverse proxy :

```bash
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2dissite 000-default.conf
sudo a2ensite kube-proxy.conf
sudo systemctl reload apache2
```

## Accès

Depuis une machine sur le réseau local :

```
http://10.0.10.xxx
```

Remplace l'adresse IP par celle de ta machine hôte (`ip a`).

## Notes

- Ne nécessite aucun LoadBalancer ni MetalLB
- Le reverse proxy Apache fait office de passerelle vers ton cluster
- Possibilité d'utiliser un nom de domaine local (apache.local) via /etc/hosts

## (Optionnel) HTTPS

Tu peux sécuriser l'accès via Let's Encrypt ou un certificat local en utilisant mod_ssl.

## Nettoyage

```bash
kubectl delete -f .
```

## 👤 Auteur

Barry
