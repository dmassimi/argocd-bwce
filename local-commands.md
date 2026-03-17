

## 1. Clean Environment
Run these commands to remove all existing ArgoCD components and previous test deployments to ensure a "clean slate."

### Delete the Application (and its associated resources)
```bash
kubectl delete application bwce-app-testing -n argocd --ignore-not-found
```

### Delete the credentials secret
```bash
kubectl delete secret my-private-repo-creds -n argocd --ignore-not-found
```

### Delete the namespaces
```bash
kubectl delete namespace bwce-test --ignore-not-found
kubectl delete namespace argocd --ignore-not-found
```
---

## 2. Install ArgoCD
Install the ArgoCD control plane into your cluster.

### Create the namespace
```bash
kubectl create namespace argocd`
```

### Apply the official ArgoCD manifests
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Wait for the pods to be ready
```bash
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=60s
```

---

## 3. Configure Private Repository Access
When using GitHub Fine-grained tokens, use 'x-access-token' as the username to ensure correct authentication.

### Create the secret with your Fine-Grained PAT
```bash
kubectl create secret generic my-private-repo-creds    --namespace argocd    --from-literal=type=git    --from-literal=url="YOUR_REPO_GIT_HERE"    --from-literal=username="x-access-token"  --from-literal=password="YOUR_TOKEN_HERE"
```

### Label the secret so ArgoCD recognizes it as a repository credential
```bash
kubectl label secret my-private-repo-creds -n argocd argocd.argoproj.io/secret-type=repository`
```

---

## 4. Retrieve Admin Credentials & Access UI
ArgoCD generates a random password for the 'admin' user during installation.

### Decode the password (PowerShell)
```bash
$pw = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($pw))
```

---

## 5. Deploy the Application
### Create secret and Apply the Application manifest
```bash
cd .\bwce-manifests
kubectl.exe apply -f .\my-license-secret.yaml -n bwce-test
cd .\argocd-apps
kubectl apply -f bwce-argocd-app.yaml -n argocd
```

### Start the Port Forward
```bash
kubectl port-forward svc/argocd-server -n argocd 8082:443
```

### Access via Browser: https://localhost:8082
### Default Username: admin

---

## 6. Post-Deployment Verification
Verify that the sync was successful and the pods are running in the target namespace.

### Check ArgoCD application status
```bash
kubectl get application bwce-app-testing -n argocd
```

### Check BWCE Pods
```bash
kubectl get pods -n bwce-test
```