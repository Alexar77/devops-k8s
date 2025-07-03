# Container Registry: GitHub Packages

## Publishing Docker Images to GitHub

1. Create a Personal Access Token  
   - Go to GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)  
   - Enable `read:packages`, `write:packages`, `delete:packages`

2. Build and push the Docker image
```bash
docker build -t ghcr.io/<GITHUB-USERNAME>/homesharing:latest -f nonroot.Dockerfile .
cat ~/github-image-repo.txt | docker login ghcr.io -u <GITHUB-USERNAME> --password-stdin
docker push ghcr.io/<GITHUB-USERNAME>/homesharing:latest
```

## Create Docker Registry Secret for Kubernetes

1. Create `.dockerconfig.json` file
```json
{
    "auths": {
        "https://ghcr.io": {
            "username": "REGISTRY_USERNAME",
            "password": "REGISTRY_TOKEN",
            "email": "REGISTRY_EMAIL",
            "auth": "BASE64_ENCODED_CREDENTIALS"
        }
    }
}
```

2. Generate the auth string
```bash
echo -n <USERNAME>:<TOKEN> | base64
```

3. Create the Kubernetes secret
```bash
kubectl create secret docker-registry registry-credentials --from-file=.dockerconfigjson=.dockerconfig.json
```

# Kubernetes Deployment

## Enable Ingress (Minikube example)
```bash
minikube addons enable ingress
```

## Deploy PostgreSQL
```bash
kubectl apply -f k8s/postgres/
```

## Deploy Spring Boot Application
```bash
kubectl apply -f k8s/spring/
```

## Setup Cert-Manager in MicroK8s
```bash
microk8s.kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.0/cert-manager.yaml
```

Once cert-manager pods are ready:
```bash
microk8s.kubectl apply -f k8s/cert/cluster-issuer.yaml
```

Certificates will be automatically created using the Ingress annotations:
```yaml
annotations:
  cert-manager.io/cluster-issuer: letsencrypt-http
  nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

# Directory Structure

```
alexar77-devops-playground/
├── README.md
├── k8s/
│   ├── cert/
│   │   └── cluster-issuer.yaml
│   ├── postgres/
│   │   ├── postgres-deployment.yaml
│   │   ├── postgres-pvc.yaml
│   │   └── postgres-svc.yaml
│   └── spring/
│       ├── spring-deployment.yaml
│       ├── spring-ingress.yaml
│       └── spring-svc.yaml
└── .devcontainer/
    ├── devcontainer.json
    └── startup_script.sh
```

# Verification Commands

```bash
kubectl get ingress
kubectl get certificate
kubectl describe certificate spring-cert
curl -v https://<YOUR-DOMAIN>
```

# Useful Links

- https://www.baeldung.com/spring-liveness-readiness-probes
- https://cert-manager.io/docs/
- https://cert-manager.io/docs/troubleshooting/
- https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-docker-registry
