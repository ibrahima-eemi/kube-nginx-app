# kube-nginx-app — Application Apache sur Kubernetes

Déploiement d'une application Apache statique dans un cluster Kubernetes avec exposition externe via LoadBalancer pour un accès depuis le réseau local.

---

## Prérequis

- Docker Desktop avec Kubernetes activé
- kubectl configuré pour accéder au cluster
- Windows PowerShell ou terminal équivalent

---

## Contenu du projet

```txt
kube-nginx-app/
├── configmap.yaml        # ConfigMap contenant le HTML personnalisé
├── deployment.yaml       # Déploiement Apache avec 2 réplicas + Health Checks
├── service.yaml          # Service LoadBalancer (expose sur port 80)
├── index.html            # Fichier HTML source
└── README.md             # Ce fichier
```

## Déploiement Kubernetes

### 1. Déployer l'application

```powershell
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Ou déployer tous les fichiers en une fois :

```powershell
kubectl apply -f .
```

### 2. Vérifier le déploiement

Vérifier que les pods sont en cours d'exécution :

```powershell
kubectl get pods
```

Résultat attendu :

```txt
NAME                         READY   STATUS    RESTARTS   AGE
apache-app-xxxxxxxxx-xxxxx   1/1     Running   0          30s
apache-app-xxxxxxxxx-xxxxx   1/1     Running   0          30s
```

### 3. Vérifier le service

```powershell
kubectl get svc apache-service
```

Résultat attendu :

```txt
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
apache-service   LoadBalancer   10.102.186.7   localhost     80:30000/TCP   5m
```

### 4. Vérifier les endpoints

```powershell
kubectl get endpoints apache-service
```

## Accès à l'application

### Accès local

L'application est accessible depuis votre machine via :

```powershell
http://localhost
```

### Accès depuis le réseau local

Pour permettre l'accès depuis d'autres machines du réseau :

1. Trouvez l'IP de votre machine Windows :

   ```powershell
   ipconfig
   ```

2. Trouvez l'IP de votre machine Linux :

    ```bash
    ifconfig
    ```

3. Les autres machines du réseau peuvent accéder via :

   ```powershell
   http://[VOTRE_IP_WINDOWS]
   ```

### Test de l'application

Testez l'accès avec curl :

```powershell
curl http://localhost
```

Réponse attendue :

```html
<!DOCTYPE html>
<html>
<head>
  <title>Bienvenue</title>
</head>
<body>
  <h1>Hello Apache2 on Kubernetes!</h1>
</body>
</html>
```

## Architecture

- **Deployment** : 2 réplicas Apache avec health checks (liveness/readiness probes)
- **ConfigMap** : Contient le fichier HTML personnalisé monté dans les containers
- **Service LoadBalancer** : Expose l'application sur localhost pour Docker Desktop
- **Ressources** : Limites CPU (0.5) et mémoire (512Mi) par pod

## Dépannage

### Pods en état Pending

Si un pod reste en état "Pending", vérifiez :

```powershell
kubectl describe pod [NOM_DU_POD]
```

### Service non accessible

Vérifiez que le service a une IP externe :

```powershell
kubectl get svc apache-service -o wide
```

### Logs des containers

Pour voir les logs d'un pod :

```powershell
kubectl logs [NOM_DU_POD]
```

## Nettoyage

Pour supprimer toutes les ressources :

```powershell
kubectl delete -f .
```

Ou individuellement :

```powershell
kubectl delete deployment apache-app
kubectl delete service apache-service
kubectl delete configmap apache2-html-config
```
