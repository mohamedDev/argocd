
# ArgoCD GitOps – ApplicationSet for Multi-Service, Multi-Environment Deployments

This repository is structured to support multiple microservices (e.g., `users-service`, `orders-service`, etc.) and multiple environments (`dev`, `qa`, `prod`) using **ArgoCD ApplicationSet**.

---

## 📁 Project Structure

```bash
apps/
├── users-service/
│   ├── dev/
│   │   ├── deployment.yaml
│   │   └── kustomization.yaml
│   ├── qa/
│   └── prod/
├── orders-service/
│   ├── dev/
│   ├── qa/
│   └── prod/
```

Each folder under `apps/<service>/<env>` contains the Kubernetes manifests or a `kustomization.yaml`.

---

## ⚙️ ArgoCD ApplicationSet

ArgoCD will automatically scan the `apps/*/*` folders and create an Application for each combination of service and environment.

### 📄 Example `ApplicationSet.yaml`

Place this file inside `argocd/` or your ArgoCD manifests directory.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: microservices-appset
  namespace: argocd
spec:
  generators:
    - git:
        repoURL: https://github.com/<your-org>/<your-repo>
        revision: HEAD
        directories:
          - path: 'apps/*/*'
  template:
    metadata:
      name: '{{path.components[1]}}-{{path.components[2]}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/<your-org>/<your-repo>
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.components[1]}}-{{path.components[2]}}'
      syncPolicy:
        automated:
          selfHeal: true
          prune: true
```

Replace `<your-org>/<your-repo>` with your actual GitHub org/repo URL.

---

## ➕ How to Add a New Service

1. Create a new folder under `apps/`:

```bash
mkdir -p apps/new-service/dev
```

2. Add your `deployment.yaml`, `service.yaml`, or `kustomization.yaml` in that folder.

3. ArgoCD will auto-detect and deploy it based on `ApplicationSet`.

---

## 🔄 How to Modify a Service

1. Edit the files inside `apps/<service>/<env>/`.
2. Push to Git.
3. ArgoCD will detect changes and sync automatically (if auto-sync is enabled).

---

## ❌ How to Remove a Service

1. Delete the corresponding folder: `apps/<service>/<env>/`
2. Commit and push.
3. ArgoCD will **prune** the deleted application from the cluster.

---

## 🧪 Requirements

- ArgoCD is installed in your cluster.
- You have `ApplicationSet` CRDs installed.
  ```bash
  kubectl get crd applicationsets.argoproj.io
  ```

---

## ✅ Benefits of this setup

- GitOps automated deployments
- Dynamic creation of apps from folder structure
- Multi-env and multi-service management at scale

---

---

## 🚀 Bootstrap ArgoCD Root Application

```bash
$ kubectl apply -f root-app.yaml
```
