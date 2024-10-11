# flask-py-db
A simple Flask Python CRUD app to understand how the basics of CRUD works with a database



# run locally 

Use the .devcontainer to run the program inside of .devcontainer.

```
version: '3.8'

services:
  app:
    image: sangam14/flask-crud:latest  # Assuming this is the image with everything set up
    restart: unless-stopped
    ports:
      - "5100:5000"  # Exposing the Flask app on port 5100 externally. Reminder the logic is HOST:CONTAINER. The container port has to be 5000 as the build image we set it as 5000 in the dockerfile.

    depends_on:
      - db

  db:
    image: postgres:latest
    restart: unless-stopped
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
      POSTGRES_PASSWORD: postgres  # Make sure to use secure passwords in production
    ports:
      - "5432:5432"

volumes:
  postgres-data:

```
## export helm chart 

Dockerfile         Pipfile            README.md          docker-compose.yml
LICENSE            Pipfile.lock       api                helm


```


### Install ArgoCD 

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```
### Port forwarding 

```
kubectl port-forward svc/argocd-server -n argocd 8080:443

Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080

```

open ArgoCD URL 
https://localhost:8080/



get the initial admin password. By default, it is stored as a secret in the ArgoCD namespace

```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

write deployement file ```flask-py-db.yaml``` 

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: python-stackgen-db
  namespace: argocd
spec:
  destination:
    namespace: default
    server: https://kubernetes.default.svc
  source:
    repoURL: https://github.com/stackgen-demo/flask-py-db
    targetRevision: HEAD  # You can specify a branch, tag, or commit hash as needed
    path: helm/python-stackgen-db  # Specify the path to the Helm chart within the repository
    helm:
      valueFiles:
        - values.yaml  # Use the values.yaml file to provide custom configurations
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

```

### deploy helm chart 

```
kubectl apply -f flask-py-db.yaml 
```

