# <p align=center>Gitops Fluxcd Project  <br> </p> 

### Table of Contents

| **SL** | **Topic** |
| --- | --- |
| 01 | [Project Overview and Requirements](#01) |
| 02 | [Introduction](#02) |
| 03 | [Tools and Technologies Used](#03) |
| 04 | [Project Structure](#04) |
| 05 | [Setup and Deployment Process](#05)  |
| 06 | [Conclusion](#06)  |

<br>

### <a name="01">Project Overview and Requirements</a>

All deployments (nginx, WordPress, MySQL, ingress-nginx, etc.) must be managed in a GitHub repository and continuously reconciled with the Kubernetes cluster using FluxCD. Nothing should be applied imperatively; everything must be done declaratively via GitOps.
<br>

**Task 1:** **Cluster Setup and Application Deployment**

●​ Create a Kubernetes cluster with k3s on your local machine.

●​ Use FluxCD (GitOps tool) to bootstrap the Flux controller into the cluster, and configure it
to sync the cluster state from a GitHub repository.

●​ Set up SOPS with age for secret encryption in FluxCD.

●​ Deploy a bare-metal load balancer solution (e.g., MetalLB or OpenELB).

●​ Deploy ingress-nginx using Helm (chart version 4.11.7) via FluxCD

●​ Configure Helm values to deploy the ingress-nginx controller as a LoadBalancer service.

●​ Deploy MySQL database, MySQL secrets (username & password) must be encrypted with SOPS & age, and stored in GitHub via FluxCD

●​ Deploy WordPress​ via FluxCD
  
**Expected Result:** After deployment, WordPress should be functional and installable at http://app.local. Make sure that WordPress can successfully connect to the MySQL database so that the installation works correctly.

<br>

**Task 2:** **Custom Ingress-NGINX Lua Plugin**

●​ Using the custom ingress-nginx Lua plugin, configure ingress-nginx to redirect all 404 (Not Found) responses to google.com.

●​ This should apply to any incoming request where the ingress-nginx resource is not
found.

**Expected Result:** Visiting http://app.local/notfound, should redirect to https://google.com

<br>

**Task 3:** **Monitoring**

●​ Deploy Prometheus, Loki, and Grafana along with Promtail (or Alloy) declaratively with FluxCD.

●​ Configure logs from Kubernetes pods (e.g., WordPress, ingress-nginx, etc.) to be sent to Loki via Promtai/Alloy.

●​ Set up a Grafana dashboard to visualize WordPress pod logs.

●​ Export the Grafana dashboard as a JSON file.

**Expected Result:** Grafana should be accessible at http://app-monitor.local

<br>

### <a name="02">Introduction</a>

This project demonstrates a complete GitOps-based deployment workflow using FluxCD on a local Kubernetes cluster. The goal is to manage all application and infrastructure resources declaratively through a GitHub repository, ensuring that the cluster state is automatically reconciled with the desired configuration stored in Git.

The setup includes deploying core components such as Nginx Ingress, MySQL, and WordPress, all managed through FluxCD. Sensitive data is securely handled using SOPS with Age encryption, and external access is managed with a bare-metal load balancer (MetalLB).

Additionally, the project implements a custom Lua plugin for ingress-nginx to handle 404 redirections and a monitoring stack with Prometheus, Grafana, and Loki to visualize logs and metrics across the cluster.

By the end of this setup, the WordPress application and monitoring dashboard should be fully operational and accessible through local domain names.

<br>

### <a name="03">Tools and Technologies Used</a>

This project is built using the following tools and technologies:
- Kubernetes
- FluxCD
- SOPS with Age
- MetalLB
- Ingress Nginx
- Helm
- WordPress
- MySQL
- Prometheus
- Grafana
- Loki
- Promtail

<br>


### <a name="04">Project Structure</a>
```
gitops-fluxcd-project
├── cluster
│   ├── apps
│   │   ├── kustomization.yaml
│   │   └── wordpress
│   │       ├── helmrelease.yaml
│   │       ├── ingress.yaml
│   │       ├── kustomization-wp.yaml
│   │       ├── kustomization.yaml
│   │       └── namespace.yaml
│   ├── database
│   │   ├── kustomization.yaml
│   │   └── mysql
│   │       ├── helmrelease.yaml
│   │       ├── kustomization-mysql.yaml
│   │       ├── kustomization.yaml
│   │       ├── mysql-secret.enc.yaml
│   │       └── namespace.yaml
│   ├── flux-system
│   │   ├── gotk-components.yaml
│   │   ├── gotk-sync.yaml
│   │   └── kustomization.yaml
│   ├── infrastructure
│   │   ├── ingress-nginx
│   │   │   ├── helmrelease.yaml
│   │   │   ├── kustomization.yaml
│   │   │   └── namespace.yaml
│   │   ├── kustomization.yaml
│   │   ├── metallb
│   │   │   ├── config
│   │   │   │   └── metallb-config.yaml
│   │   │   ├── helmrelease.yaml
│   │   │   ├── kustomization-metallb.yaml
│   │   │   ├── kustomization.yaml
│   │   │   └── namespace.yaml
│   │   └── monitoring
│   │       ├── grafana
│   │       │   ├── dashboards
│   │       │   │   ├── dashboard-ingress-logs.json
│   │       │   │   └── dashboard-wordpress-logs.json
│   │       │   ├── helmrelease.yaml
│   │       │   ├── ingress-dashboard-configmap.yaml
│   │       │   ├── ingress.yaml
│   │       │   ├── kustomization.yaml
│   │       │   └── wordpress-dashboard-configmap.yaml
│   │       ├── kustomization.yaml
│   │       ├── loki
│   │       │   ├── helmrelease.yaml
│   │       │   └── kustomization.yaml
│   │       ├── namespace.yaml
│   │       ├── prometheus
│   │       │   ├── helmrelease.yaml
│   │       │   └── kustomization.yaml
│   │       └── promtail
│   │           ├── helmrelease.yaml
│   │           └── kustomization.yaml
│   └── sources
│       ├── bitnami-helmrepo.yaml
│       ├── grafana-helmrepo.yaml
│       ├── ingress-nginx-helmrepo.yaml
│       ├── kustomization.yaml
│       └── prometheus-helmrepo.yaml
├── plain-secret
│   └── mysql-plain-secret.yaml
└── screenshots
└── README.md
```

<br>

### <a name="05">Setup and Deployment Process</a>

**1. Prerequisites**

```sudo apt install -y curl git ufw vim wget gnupg apt-transport-https```

```sudo swapoff -a```

```sudo sed -i '/ swap / s/^/#/' /etc/fstab```


**2. Install and configure k3s**

```curl -sfL https://get.k3s.io | sh -```

```mkdir -p $HOME/.kube```

```vim ~/.bashrc```

```export KUBECONFIG=$HOME/.kube/config```

```source ~/.bashrc```

```sudo cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config```

```sudo chown $(id -u):$(id -g) $HOME/.kube/config```

```kubectl get nodes```

The output will be:
```
NAME          STATUS   ROLES                  AGE   VERSION
shadikul-pc   Ready    control-plane, master   18d   v1.33.4+k3s1
```

**3. Install Flux CLI**

```curl -s https://fluxcd.io/install.sh | sudo bash```

```flux --version```


**4. Install SOPS**

```curl -LO https://github.com/getsops/sops/releases/download/v3.10.2/sops-v3.10.2.linux.amd64```

```sudo mv sops-v3.10.2.linux.amd64 /usr/local/bin/sops```

```sudo chmod +x /usr/local/bin/sops```


**5.  Install Age**

```wget https://github.com/FiloSottile/age/releases/download/v1.2.1/age-v1.2.1-linux-amd64.tar.gz```

```tar -xvf age-v1.2.1-linux-amd64.tar.gz```

```sudo mv age/age age/age-keygen /usr/local/bin/```

```age-keygen --version && age --version```

```mkdir -p $HOME/.config/sops/age```

```age-keygen -o $HOME/.config/sops/age/keys.txt```

```cat $HOME/.config/sops/age/keys.txt```

<img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/1-age-key.png"> <br>


**6. Git Repository Setup**

Follow these steps to prepare your Git repository for FluxCD:

- Create a new Git repository on GitHub to store all your Kubernetes manifests and configuration files.

- Clone the repository to your local machine.

- Download or clone this repository that contains all the preconfigured files.

- Move the **cluster** and **plain-text** folders from the downloaded repository into your newly created Git repository local folder.

- Delete the flux-system folder from your local repository folder to avoid conflicts with your own Flux bootstrap process.


**7. Encrypt MySQL Secret with SOPS and Age**

```sops --encrypt --age $(grep 'public key:' $HOME/.config/sops/age/keys.txt | awk '{print $4}') --encrypted-regex '^(data|stringData)$' plain-secret/mysql-plain-secret.yaml > cluster/database/mysql/mysql-secret.enc.yaml```

After running that command, it will create a file named **mysql-secret.enc.yaml** inside the **cluster/database/mysql/** directory.

Decrypt the encrypted file and compare it with the plain-text version to ensure both contain the same values.

```cat plain-secret/mysql-plain-secret.yaml && sops -d cluster/database/mysql/mysql-secret.enc.yaml```

<img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/2-sops-secret.png"> <br>


**8. Prepare Kubernetes Secrets**

- Create a Docker Hub Secret

- Go to Docker Hub and create an account (if you don’t already have one).

- Generate an access token from your Docker Hub account settings.

- Use this token to create a Kubernetes secret for pulling images from Docker Hub.

```
kubectl create namespace flux-system
```
```
kubectl get ns
```
```
kubectl create secret docker-registry dockerhub-cred \
  --docker-server=registry-1.docker.io \
  --docker-username=shadikul \
  --docker-password=your-docker-hub-token \
  --docker-email=shadikul.islam.shuvo@gmail.com \
  -n flux-system
```
- Create a Kubernetes secret that stores the Age private key, which FluxCD will use to decrypt SOPS-encrypted files.
```
kubectl create secret generic sops-age-key \
  --from-file=age.agekey=$HOME/.config/sops/age/keys.txt \
  -n flux-system
```


**9. Connect FluxCD with GitHub Repository**

- Create a GitHub Personal Access Token

  - Log in to your GitHub account
  - Go to Settings → Developer settings → Personal access tokens → Tokens (classic).
  - Click Generate new token and grant the necessary permissions (typically repo and workflow).

- Copy the generated token securely. You’ll need it to authenticate with GitHub from your terminal.

- Run flux bootstrap GitHub command:
  
```
flux bootstrap github \
  --owner=Shadikul-Islam \
  --repository=gitops-fluxcd-project \
  --branch=master \
  --path=cluster \
  --personal
```

**10. Apply One-Time Custom Kustomizations**

- Apply the following two custom Kustomizations. These steps are only required once for the cluster setup:

```kubectl apply -f cluster/database/mysql/kustomization-mysql.yaml```

```kubectl apply -f cluster/infrastructure/metallb/kustomization-metallb.yaml```


<br>

### <a name="05">Deployment Verification</a>

The following screenshots demonstrate that all components of the project have been successfully deployed and are functioning as expected:

- Kubernetes Components Verification

  ```kubectl get ns```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/3-ns.png"> <br>

  ```kubectl get all -n flux-system```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/4-flux-system.png"> <br>

  ```kubectl get all -n ingress-nginx```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/5-ingress.png"> <br>

  ```kubectl get all -n metallb-system```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/6-metallb.png"> <br>

  ```kubectl get all -n monitoring```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/7-monitoring.png"> <br>

  ```kubectl get all -n mysql```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/8-mysql.png"> <br>

  ```kubectl get all -n wordpress```
  
  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/9-wordpress.png"> <br>

  ```flux get all```
  
  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/18-flux.png"> <br>

  ```kubectl get ingress --all-namespaces```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/10-ingress.png"> <br>

- Add the following entries to your ```/etc/hosts``` file to map the local domains to your cluster’s load balancer IP
  
  ```sudo vim /etc/hosts```

  ```192.168.26.227 app.local```
  
  ```192.168.26.227 app-monitor.local```

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/17-etc-hosts.png"> <br>

- WordPress Application Accessible at http://app.local and Admin Panel Accessible at http://app.local/wp-admin

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/11-wp-app.png"> <br>

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/12-wp-admin.png"> <br>

- Monitoring Dashboard Grafana is accessible at http://app-monitor.local. **Username:** admin and **Password:** admin. These credentials are declared in the ```cluster/infrastructure/monitoring/grafana/helmrelease.yaml``` file.

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/13-grafana.png"> <br>

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/14-dashboard-1.png"> <br>

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/15-dashboard-2.png"> <br>

  <img src= "https://github.com/Shadikul-Islam/gitops-fluxcd-project/blob/master/screenshots/16-dashboard-3.png"> <br>

<br>

### <a name="06">Conclusion</a>

This project successfully delivers a fully automated, GitOps-driven Kubernetes environment using FluxCD. All resources, including infrastructure, applications, secrets, and monitoring, are managed declaratively through a GitHub repository, ensuring consistency and traceability across deployments.

The custom ingress-nginx Lua plugin effectively handles 404 redirections, while the integrated monitoring stack (Prometheus, Loki, Grafana) provides complete visibility into cluster health and application logs.

Overall, this implementation demonstrates how to build a production-style, end-to-end GitOps workflow that combines automation, security, and observability in a local Kubernetes environment.

