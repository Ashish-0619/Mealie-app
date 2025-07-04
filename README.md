# 🍽️ Mealie Deployment with Helmfile (Rancher Desktop + Traefik)

This project deploys the [Mealie](https://github.com/mealie-recipes/mealie) recipe management app to a local Kubernetes cluster using **Helm**, **Helmfile**, and **Traefik Ingress** (default in Rancher Desktop).

---

## 📦 Prerequisites

Make sure the following are installed:

- [Rancher Desktop](https://rancherdesktop.io/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [helm](https://helm.sh/)
- [helmfile](https://github.com/helmfile/helmfile)

---

## 📁 Project Structure

```bash
mealie/
├── charts/
│   └── mealie/                 # Helm chart for Mealie
│       ├── templates/
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   └── ingress.yaml
│       ├── Chart.yaml
│       └── values.yaml
├── helmfile.yaml               # Helmfile config
└── README.md                   # You're here!

```
---

## 🚀 Installation & Deployment

### 1. Clone the Repository

```bash
git clone https://github.com/<your-username>/mealie-helm.git
cd mealie-helm
```
Create the Namespace
```bash
kubectl create namespace mealie
```
3. Deploy with Helmfile
```bash
helmfile apply
``

🌍 Access the Application


Option 1: Update /etc/hosts

Add the following entry to your system's /etc/hosts file:

127.0.0.1 mealie.localhost

Then access the app in your browser at:

http://mealie.localhost:30080

    30080 is the default Traefik NodePort for HTTP traffic in Rancher Desktop.

Option 2: Port Forward (Alternative)

If needed, use:

kubectl port-forward -n mealie svc/mealie 9000:9000

Then visit:

http://localhost:9000


🧹 Cleanup / Uninstall

helmfile destroy
kubectl delete namespace mealie


🙌 Credits

    Mealie

    Helm

    Helmfile

    Traefik

    Rancher Desktop
