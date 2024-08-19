# Installing SUSE Rancher In An Air-Gapped

This repo is created to provide the reader all the required information on install `SUSE Rancher` using the Air-Gapped Manual method. This repo provide a high-level explanation along with install and configure step-by-step guides and scripts.

---

<p align="center">
    <img src="Images/Rancher-Logo.png">
</p>

---

## About This Repo

`SUSE` recommends using Helm, a Kubernetes package manager, to install `SUSE Rancher`. This is the main recommended method of installing and deploying Rancher in your environment. Even in an Air-Gapped environment, Helm is also the preferred method for deploying `SUSE Rancher`, however, in an Air-Gapped environment, all required files and images must be downloaded on a JumpBpx with internet access and then move files to the Kubernetes Cluster nodes and images added to the private local registry. 

`SUSE Rancher` also supports the installation behind an HTTP proxy, if any exists in your environment, however, some extra configurations is required when deploying and installing `SUSE Rancher`. These configuration is also going to be passed thorough the Helm Chart Values.

`SUSE Rancher` has officially provided a Helm Chart Repository to be used when deploying and installing `SUSE Rancher`. The available `SUSE Rancher` Helm Charts repositories can be found in [this link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster#1-add-the-helm-chart-repository)

`SUSE Rancher` Depends on `cert-manager` for certificate management, thus, it is also required to deploy and install `cert-manager` while installing `SUSE Rancher`. `cert-manager` is also recommended to be deployed and installed using Helm Chart. The Helm Chart Repository for `cert-manager` can be found at https://charts.jetstack.io

This repo is designed and created to provide a documented step-by-step guide and scripts for installing `SUSE Rancher` using Air-Gapped method. Using this method, Internet access is not required for the nodes, however, it is required for the JumpBox or Laptop that will be used to download the required files and images.

---

## Step-By-Step Guide

This section is a ste-by-step guide for installing the `SUSE Rancher` using the Air-Gapped method with the Helm CLI tool.

---

### Prerequisites For Installing SUSE Rancher

---

Before installing `SUSE Rancher`, you must confirm that all requirements are in place to avoid any issues during or after the installation.

> Ensure Using A Supported OS

All supported operating systems are 64-bit x86. `SUSE Rancher should` work with any modern Linux distribution. The `SUSE Rancher` support matrix - [link here](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-8-6/) - lists which OS and Docker versions were tested for each Rancher version.

> Ensure Using A Supported Kubernetes Distribution

`SUSE Rancher` needs to be installed on a supported Kubernetes version. `SUSE Rancher` supports several kubernetes distribution and versions of those kubernetes distributions. For the list of supported kubernetes distributions and the associated versions, please refer to the `SUSE Rancher` support matrix - [link here](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-8-6/) - to ensure that your intended version of Kubernetes is supported.

> IF Using K3S Kubernetes Distribution, Ensure Specific Requirements Are In Place

If you are deploying `SUSE Rancher` on `SUSE Rancher K3S` cluster, several specific requirements are required. Please refer to this [link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements#k3s-specific-requirements) for the list of specific requirements needed

> Ensure Sufficient Hardware Resources

`SUSE Rancher's` hardware footprint depends on a number of factors, including:
- Size of the managed infrastructure (e.g., node count, cluster count).
- Complexity of the desired access control rules (e.g., RoleBinding object count).
- Number of workloads (e.g., Kubernetes deployments, Fleet deployments).
- Usage patterns (e.g., subset of functionality actively used, frequency of use, number of concurrent users).

Thus, The hardware resources requirements may vary, `SUSE` have officially provided recommended hardware resources requirements as a starting point:
- For hardware requirements when using RKE2 as a kubernetes distribution, please refer to this [link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements#rke2-kubernetes)
- For hardware requirements when using K3S as a kubernetes distribution, please refer to this [link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements#k3s-kubernetes)
- For hardware requirements when using Hosted Kubernetes (EKS, GKE, AKS) as a kubernetes distribution, please refer to this [link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements#hosted-kubernetes)

> Ensure Proper Disk Requirements Are In Place

`SUSE Rancher` performance depends on etcd in the cluster performance. To ensure optimal speed, we recommend always using SSD disks to back your `SUSE Rancher` management Kubernetes cluster. On cloud providers, you will also want to use the minimum size that allows the maximum IOPS. In larger clusters, consider using dedicated storage devices for etcd data and wal directories.

> Disable Firewalld Services On Node(s)

Some distributions of Linux may have default firewall rules that block communication within the Kubernetes cluster. Since Kubernetes v1.19, firewalld must be turned off, because it conflicts with the Kubernetes networking plugins. To do so, please use the following command:

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

If you don't feel comfortable doing so, you might check other suggestions. Some users were successful creating a separate firewalld zone with a policy of ACCEPT for the Pod CIDR. To do so, please refer to this [link](https://github.com/rancher/rancher/issues/28840#issuecomment-787404822)

> Ensure Node IP Is Static Assignments Or DHCP Have Reservation

Each node used (where `SUSE Rancher` will be deployed on) should have a static IP configured, regardless of whether you are installing Rancher on a single node or on an HA cluster. In case of DHCP, each node should have a DHCP reservation to make sure the node gets the same IP allocated.

> Ensure Proper Node Networking And Communication To Downstream Clusters

`SUSE Rancher` is a kubernetes clusters management solution, thus, `SUSE Rancher` will need to communication with downstream clusters to be able to manage them. Thus, proper networking and routing configuration from the nodes(s) (where `SUSE Rancher` will be deployed on) to the downstream cluster is required.

Also, `SUSE Rancher` may integrate with external tools such as external authentication provider (LDAP, AD, etc.). Thus, proper networking and routing configuration from the nodes(s) (where `SUSE Rancher` will be deployed on) to these external tools is required.

> Ensure NTP Is Set Properly

The ntp (Network Time Protocol) package should be installed. This prevents errors with certificate validation that can occur when the time is not synchronized between the client and server.

> Ensure Require Network Ports Are Opened On Firewall (If Any Exists)

To operate properly, `SUSE Rancher` requires a number of ports to be open on `SUSE Rancher` node(s) (where `SUSE Rancher` will be deployed on) and on downstream Kubernetes cluster nodes. Port Requirements lists all the necessary ports for Rancher and Downstream Clusters for the different cluster types.

> Ensure A JumpBox Server Is In Place With Sufficient Storage Capacity

As this deployment is based on the Air-Gapped method (no internet access to the node(s)), we would need to download several files and images to a JumpBox server which have internet access. The JumpBox server could be of any operating system. The JumpBox will require at least 350GB of disk space or more available. The JumpBox must have docker CLI tool (or any other alternatives) installed to be able to communicate, pull and push images to the image registries(s). The JumpBox also required Helm CLI toll to be installed.

JumpBox Requirements:
- CPU: recommended min of 8 CPU - this is not a must, however, this will speed up the process
- Memory: recommended min of 32GB RAM - this is not a must, however, this will speed up the process
- Disk: 300 GB - this is a must, otherwise the JumpBox will not have enough space to store the images and create the required Tarball
- Tools:
  - Docker installed and enabled
  - Helm installed and enabled
  - Kubectl installed and enabled with the kubeconfig of the management cluster available for kubectl connection

> Ensure A Local Image Registry Is In Place With Sufficient Storage Capacity

As this deployment is based on the Air-Gapped method (no internet access to the node(s)), we would need to download several files and images to the JumpBox server (which have internet access). Once this is completed, we will be uploading the images to a local Image Registry. Thus, a local image registry is required. There is no specific Image Registry required, any OCI Image Registry tool will be enough. The local Images Registry will require at least 350GB of disk space or more available.

Harbor Requirements:
- CPU: recommended min of 4 CPU - this is not a must, however, this will speed up the process
- Memory: recommended min of 16GB RAM - this is not a must, however, this will speed up the process
- Disk: 300GB - this is a must, otherwise the upload will not have enough space

---

### Downloading The Required Files And Images To The JumpBox

---

As this deployment is based on the Air-Gapped method (no internet access to the node(s)), we would need to download several Helm Charts, CRDs (Customer Resources Definitions), files, and images to the JumpBox server (which have internet access). Once this is completed, we will be uploading the images to the local Image Registry (Thus you need to have a local image registry already available in your environment). After uploading the required images to the local Image Registry, we will then move the required Helm Charts, CRDs, and files from the JumpBox to the node(s) to start the Air-Gapped Deployment.

> Helm Charts

The recommended way of deploying `SUSE Rancher` is through using Helm, even in Air-Gapped situation. Also, `SUSE Rancher Depends on Cert-Manager for certificate generation. Cert-Manager is also recommended to be deployed using Helm. Thus, several Helm charts are required to be downloaded

The required Helm Repositories to be added to the JumpBox so that we can retrieve the required Helm Charts are:
- JetStack [For Cert-Manager] - URL: https://charts.jetstack.io
- `SUSE Rancher` provides several Helm Charts Repository, please chose one of the below (Recommended Stable Option):
  - `SUSE Rancher` Latest [Recommended for trying out the newest features] - URL: https://releases.rancher.com/server-charts/latest
  - `SUSE Rancher` Stable [Recommended for trying out the newest features] - URL: https://releases.rancher.com/server-charts/stable
  - `SUSE Rancher` Alpha [Recommended for trying out the newest features] - URL: https://releases.rancher.com/server-charts/alpha
    - Please Note: The above mentioned `SUSE Rancher` Helm repositories are for the community edition, for Rancher Prime Repository, please refer to `SUSE` representative to retrieve it.

To add the above mentioned Helm Repositories, please use the below commands:
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add rancher https://releases.rancher.com/server-charts/stable
helm repo update
```

> Files

`SUSE Rancher` have officially provided several files that can help in the Air-Gapped deployment. These files are:
- rancher-images.txt
  - This file will include all required `SUSE Rancher` images name to be able to pull them from the internet
- rancher-load-images.sh
  - This is a script that will help loading all required images in to the private registry
- rancher-save-images.sh
  - This is a script that will help saving all of the images needed for `SUSE Rancher` to a tarball file

You can download these files from the `SUSE Rancher` official GitHub. For the release page, please refer to this [link](https://github.com/rancher/rancher/releases). Chose the release you would like to deploy, go to the release page itself (this [link](https://github.com/rancher/rancher/releases/tag/v2.8.5) is an example for `SUSE Rancher` version 2.8.5) and then scroll down to the end of the page where you will be able to find al the assets including the above-mentioned files.

> Images

To deploy `SUSE Rancher`, several images is required to be downloaded and pushed to the local images registry. The above-mentioned file and scripts in the files sub-section will help to pull and push the required images. 

`SUSE Rancher` required Cert-Manager to be deployed to handel the certificate generation of the `SUSE manager` solution. We will also need to download the required images and push it to the local image registry

To do so, we will use the Helm Cli tool to perform this action. Please follow the below steps:
- fetch the cert-manager helm chart using Helm Cli tool - This can only be done after you have added the jetstack Helm repository and updated it.
```bash
helm fetch jetstack/cert-manager --version <required-version-if-needed>
```
- Get all required images of this Helm chart using Helm template command and export them to the rancher-images.txt
```bash
helm template ./cert-manager-<version>.tgz | awk '$1 ~ /image:/ {print $2}' | sed s/\"//g >> ./cert-manager-images.txt
```

Once we have the images name, we will use docker to pull and push the images

---

==========================================================================

### Install SUSE Rancher In An Air-Gapped Environment - Step-By-Step Guide

==========================================================================

---

In this guide, we will be deploying an `SUSE Rancher` on a single node RKE2 cluster. For production environment, it is required to deploy `SUSE Rancher os a highly-available cluster with minimum of 3 nods. For a high-available RKE2 cluster, please refer to [this link](https://docs.rke2.io/install/ha)

1. Ensure you have performed and satisfied all prerequisites mentioned [here](#prerequisites-for-installing-suse-rancher)
2. On your local image registry, create a repository called rancher (repository name must be rancher). In this example i have created my repository on harbor image registry and mad it public to avoid the authentication requirement.

<p align="center">
    <img src="Images/step-2.png">
</p>

3. SSH to the JumpBox (Or use your laptop) and add the required Helm repositories. Please note, the below command will add the `SUSE Rancher` community edition Helm repository. If you are deploying the `SUSE Rancher Prime`, which is the enterprise edition of `SUSE Rancher`, a different Helm repository URL is require. To get this URL, please refer to your SUSE representative. Please Note: as the process is going to take several hours, if the ssh terminal session got idle for too long and halted, problems may occur, thus it is recommended to set the timers for the session to be as log as possible. In this guide i am using the -o "ServerAliveInterval 120" -o "ServerAliveCountMax 240" options in the SSH command to ensure the session will run for 4 hours.

SSH Command:
```bash
ssh -i <key>.pem -o "ServerAliveInterval 120" -o "ServerAliveCountMax 240" <user>@<ip>
```

Commands to add the Helm Repos:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add rancher https://releases.rancher.com/server-charts/stable
helm repo update
```

<p align="center">
    <img src="Images/step-3.png">
</p>

Please Note: The above `SUSE Rancher` Helm Chart Repository is for the community edition. If you are looking to deploy `SUSE Rancher Prime` which is the enterprise edition, you need to use a different URL for the `SUSE Rancher Prime` Helm Repositories. To retrieve the `SUSE Rancher Prime` Helm Repositories, please refer to your `SUSE` representative.

4. Create a folder called `rancher-artifacts` then download in it the cert-manager Helm chart using Helm Fetch command, then retrieve all required images of the Helm cart using the helm template command and dump it in to a file called cert-manager-images.txt. With the Helm fetch command, you can specify a specific version, in this example i am using version 1.13.0. Also, we will download the cert-manager CRDs (Custom Resource Definitions) and use the same version

```bash
mkdir rancher-artifacts
cd rancher-artifacts/
helm fetch jetstack/cert-manager --version 1.13.0
helm template ./cert-manager-v1.13.0.tgz | awk '$1 ~ /image:/ {print $2}' | sed s/\"//g >> ./cert-manager-images.txt
curl -L -o cert-manager-crd.yaml https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.crds.yaml
```

<p align="center">
    <img src="Images/step-4.png">
</p>

5. Using Docker, pull all the images listed in the cert-manager-images.txt
```bash
cat cert-manager-images.txt
docker pull quay.io/jetstack/cert-manager-cainjector:v1.13.0
docker pull quay.io/jetstack/cert-manager-controller:v1.13.0
docker pull quay.io/jetstack/cert-manager-webhook:v1.13.0
docker pull quay.io/jetstack/cert-manager-ctl:v1.13.0
```

<p align="center">
    <img src="Images/step-5.png">
</p>

6. Using Docker, Login to your local image registry, tag the pulled images with your local image registry URL and the rancher repository (also remove the first 2 parts of the images URLs), then push them to the local image registry

```bash
docker login suse-harbor-poc.suse-rancher-demo.com
docker tag quay.io/jetstack/cert-manager-cainjector:v1.13.0 suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-cainjector:v1.13.0
docker tag quay.io/jetstack/cert-manager-controller:v1.13.0 suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-controller:v1.13.0
docker tag quay.io/jetstack/cert-manager-webhook:v1.13.0 suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-webhook:v1.13.0
docker tag quay.io/jetstack/cert-manager-ctl:v1.13.0 suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-ctl:v1.13.0
docker push suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-cainjector:v1.13.0
docker push suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-controller:v1.13.0
docker push suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-webhook:v1.13.0
docker push suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-ctl:v1.13.0
```

<p align="center">
    <img src="Images/step-6-1.png">
</p>

Check that the images was pushed ot the local image registry properly

<p align="center">
    <img src="Images/step-6-2.png">
</p>

7. Download the `SUSE Rancher` Helm Chart using Helm Fetch command. In this guide we will be deploying `SUSE Rancher` version 2.8.5.

```bash
helm fetch rancher/rancher --version=v2.8.5
```

<p align="center">
    <img src="Images/step-7.png">
</p>

8. Download all required `SUSE Rancher` files related to the required release. In this guide we will be deploying `SUSE Rancher` version 2.8.5. To download the required file for this release, please refer to this [link](https://github.com/rancher/rancher/releases/tag/v2.8.5). The files to be downloaded are rancher-images.txt, rancher-load-images.sh, and rancher-save-images.sh.

<p align="center">
    <img src="Images/step-8-1.png">
</p>

```bash
cd /rancher-artifacts
wget https://github.com/rancher/rancher/releases/download/v2.8.5/rancher-images.txt
wget https://github.com/rancher/rancher/releases/download/v2.8.5/rancher-load-images.sh
wget https://github.com/rancher/rancher/releases/download/v2.8.5/rancher-save-images.sh
```

<p align="center">
    <img src="Images/step-8-2.png">
</p>

At this point, you should have the below files located in the rancher-artifacts directory
- cert-manager-v1.13.0.tgz  - Cert-Manager Helm Chart
- cert-manager-crd.yaml  - Cert-Manager CRDs
- cert-manager-images.txt
- rancher-2.8.5.tgz  - SUSE Rancher Manager Helm Chart
- rancher-images.txt 
- rancher-load-images.s
- rancher-save-images.sh

<p align="center">
    <img src="Images/step-8-3.png">
</p>

9. Save the images to a Tarball using the rancher-save-images.sh script. 
- Please Note: 
  - This process may take several hours to complete depending on the hardware.
  - The downloaded images may require more than 100GB of storage.
  - The compressed tarball will be 20-40GB in size.

```bash
# Make Script Executable
chmod +x rancher-save-images.sh
# Run Script
./rancher-save-images.sh -l rancher-images.txt -i rancher-images.tar.gz
```

Please Note: the above commands will download the Rancher Community edition from the public GitHub registry, if you are deploying `SUSE Rancher Prime` which is the enterprise edition, you ned to add to the rancher-save-image.sh command the -s option with the official `SUSE Rancher prime` registry (`./rancher-save-images.sh -l rancher-images.txt -i rancher-images.tar.gz -s <suse-rancher-prime-registry.com>`). To get the official Rancher Prime registry, please reach out to you `SUSE` representative.

```bash
# Make Script Executable
chmod +x rancher-save-images.sh
# Run Script
./rancher-save-images.sh -l rancher-images.txt -i rancher-images.tar.gz -s <suse-rancher-prime-registry.com>
```

<p align="center">
    <img src="Images/step-9-1.png">
</p>

After the script is finished, this will be the output. Please note, this process alone took over 3 hours and the files created is nearly 38GB of size. also the full storage consumed is nearly 300GB.

<p align="center">
    <img src="Images/step-9-2.png">
</p>

10. Login to your local private registry and then use the rancher-load-images.sh scrip to push the images to the image registry
- Please Note:
  - This process may take more several hours to complete depending on hardware.
  - The downloaded images may require more than 100GB of storage on the private registry machine.

**Please Note:** In a normal situation the JumpBox do not have full proper access to the Air-Gapped environment, this the scripts is designed to perform all the mentioned actions and create the Tarball and then move these files into the Air-Gapped environment. So in this situation, you should first scp all the files in the rancher-artifacts directory to a server in the Air-Gapped environment and then continue from the Air-Gapped Environment. However, in this Guide i will continue using the same JumpBox as i have full access to the Air-Gapped Environment. 

**So, Form this point forward, when a JumpBox is mentioned, it is pointing to the Server that have full access to the Air-Gapped environment - whether this server is the same JumpBox you have been using from the beginning or another server within the Air-Gapped environment where you have copied all required files to and have full access to the Air-Gapped environment**

The files to be moved to the server in the Air-Gapped Environment are:
- cert-manager-v1.13.0.tgz  - Cert-Manager Helm Chart
- cert-manager-crd.yaml  - Cert-Manager CRDs
- cert-manager-images.txt
- rancher-2.8.5.tgz  - SUSE Rancher Manager Helm Chart
- rancher-images.tar.gz  - Tarball created by the rancher-save-images.sh script containing all images - Please Note: this file is around 20 - 40GB in size
- rancher-images.txt 
- rancher-load-images.s
- rancher-save-images.sh
  
```bash
# Login to local private registry
docker login suse-harbor-poc.suse-rancher-demo.com
# Make Script Executable
chmod +x rancher-load-images.sh
# Run Script
./rancher-load-images.sh -l rancher-images.txt -i rancher-images.tar.gz -r suse-harbor-poc.suse-rancher-demo.com
```

Please Note: the above commands will download the Rancher Community edition from the public GitHub registry, if you are deploying `SUSE Rancher Prime` which is the enterprise edition, you ned to add to the rancher-load-images.sh command the -s option with the official `SUSE Rancher prime` registry (`./rancher-load-images.sh -l rancher-images.txt -i rancher-images.tar.gz -r suse-harbor-poc.suse-rancher-demo.com -s <suse-rancher-prime-registry.com>`). To get the official Rancher Prime registry, please reach out to you `SUSE` representative.

```bash
# Login to local private registry
docker login suse-harbor-poc.suse-rancher-demo.com
# Make Script Executable
chmod +x rancher-load-images.sh
# Run Script
./rancher-load-images.sh -l rancher-images.txt -i rancher-images.tar.gz -r suse-harbor-poc.suse-rancher-demo.com -s <suse-rancher-prime-registry.com>
```

<p align="center">
    <img src="Images/step-10-1.png">
</p>

After the script is completed, please check that all images are uploaded to the private local image registry properly (check the number of images). Please note, this process alone took over 3 hours and the files created is nearly 38GB of size. also the full storage consumed is nearly 300GB.

<p align="center">
    <img src="Images/step-10-2.png">
</p>

11. Make sure you have proper kubectl connection from the current JumpBox to the management cluster where the `SUSE Rancher` will be deployed on. You may need to retrieve the kubeconfig from the management cluster and add it to the current JumpBpx and then configure the current JumpBpx to use this kubeconfig file.

```bash
kubectl get nodes
```

<p align="center">
    <img src="Images/step-11.png">
</p>

12. From the current JumpBox using kubectl and helm to create a namespace and install cert-manager CRDs and the application itself. Please note: the local image repository used for this do not have authentication, if there is please add another value in the cert-manager Helm chart pointing to a secret with the authentication required (--set imagePullSecrets=regcred). Please Note: the nodes in the management cluster that `SUSE Rancher` will be deployed has to be logged in to the private local image registry. 

```bash
kubectl create namespace cert-manager
kubectl apply -f cert-manager-crd.yaml # You have to be in the rancher-artifacts directory
helm install cert-manager ./cert-manager-v1.13.0.tgz \
  --namespace cert-manager \
  --set image.repository=suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-controller \
  --set webhook.image.repository=suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-webhook \
  --set cainjector.image.repository=suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-cainjector \
  --set startupapicheck.image.repository=suse-harbor-poc.suse-rancher-demo.com/rancher/cert-manager-ctl 
```

<p align="center">
    <img src="Images/step-12-1.png">
</p>

You can check the deployment using `kubectl -n cert-manager get all`

<p align="center">
    <img src="Images/step-12-2.png">
</p>

13. Install the `SUSE Rancher` using Helm. Please note: in this guide we are using self-signed certificate. In order to use a signed certificate, please refer to this [link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/resources/add-tls-secrets)

```bash
helm install rancher ./rancher-2.8.5.tgz \
  --namespace cattle-system \
  --create-namespace \
  --set replicas=1 \
  --set hostname=suse-rancher-poc.suse-rancher-demo.com \
  --set useBundledSystemChart=true \
  --set rancherImage=suse-harbor-poc.suse-rancher-demo.com/rancher/rancher \
  --set systemDefaultRegistry=suse-harbor-poc.suse-rancher-demo.com/rancher \
  --set bootstrapPassword=SusePoc@123
```

<p align="center">
    <img src="Images/step-13-1.png">
</p>

You can confirm the deploying using the below commands

```bash
kubectl get namespace
kubectl -n cattle-system get all
```

Now you can login to your `SUSE Rancher` UI

<p align="center">
    <img src="Images/step-13-3.png">
</p>

---

## References

- [SUSE Rancher Installation Requirements](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements)
- [SUSE Rancher Support Matrix](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-8-6/)
- [SUSE Rancher Air-Gapped Deployment With Helm CLI](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install)
- [SUSE Rancher Required Images For Air-Gapped Install](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/publish-images)
- [Install SUSE Rancher](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/install-rancher-ha)
- [SUSE Rancher GitHub Release Page](https://github.com/rancher/rancher/releases)
- [SUSE Rancher Adding TLS Secret](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/resources/add-tls-secrets)


---

**Enjoy** :blush:
