# Installing SUSE NeuVector Using Kubernetes YAML File

This repo is created to provide the reader all the required information on install `SUSE NeuVector` using Kubernetes YAML File . This repo provide a high-level explanation along with install and configure step-by-step guides and scripts.

---

<p align="center">
    <img src="Images/NeuVector-Logo.png">
</p>

---

## About This Repo

`SUSE NeuVector` provides several method of deployment with the easiest of them is through the `SUSE Rancher`. However, `SUSE NeuVector` can also be installed using Kubernetes YAML files, ConfigMaps, Operator, and Helm Chart. In this repo, we will be deploying `SUSE NeuVector` using Kubernetes YAML files.

`SUSE NeuVector` components are Manager, Controller, Enforcer, Scanner, and Updater. Each components is deployed separably in different PODs, however, there is an option to deploy all components (Manager, Controller, and Enforcer) called AllInOne. However, the AllInOne is not aimed for production environment and only for testing purposes.For more info regarding the AllInOne deployment, please refer to this [link](https://open-docs.neuvector.com/special)

You can use Kubernetes to deploy separate manager, controller and enforcer containers and make sure that all new nodes have an enforcer deployed. NeuVector requires and supports Kubernetes network plugins such as flannel, weave, or calico.

The NeuVector components are deployed (in all deployment methods) by default in the following format:
- Manager - Deployed as a POD - by default 1 POD
- Controller - Deployed as a POD - by default 3 PODs - Require Odd numbers for cluster formation and leader election - 3, 5, 7, ...
- Scanner - Deployed as a POD - by default 2 PODs - can be set to be auto-scaled
- Enforcer - Deployed as a DaemonSet - by default 1 POD per node.
- Updater - Deployed as a CronJob - runs everyday at midnight.

By default, these components will be scheduled on all nodes within the cluster, however, you can use labels to direct the deployment of the Manager, Controller, and Scanner to specific nodes.

Only the Controller POD would require a PersistentVolume to store data into it. The PersistentVolume defined in the cluster is required for persistent volume support. The requirement for NeuVector is that the accessModes needs to be ReadWriteMany(RWX). Once the PersistentVolume is created, there needs to be created a PersistentVolumeClaim. Currently the persistent volume is used only for the NeuVector configuration backup files in the controller (Policies, Rules, user data, integrations etc) and registry scan results.

NeuVector Provides a sample YAML file that can be used to deploy the `SUSE NeuVector`. You can find the sample file at this [link](https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/neuvector-k8s.yaml). Also available in the [YAML-File Directory](/1-Install/NeuVector/2-Install-Neuvector-With-Kubernetes-Yaml/YAML-Files/NeuVector-Kubernetes-Sample.yaml) of this repo.

The sample file will deploy one manager and 3 controllers. It will deploy an enforcer on every node as a daemonsets. See the bottom section for specifying dedicated manager or controller nodes using node labels. Note: It is not recommended to deploy (scale) more than one manager behind a load balancer due to potential session state issues. If you plan to use a PersistentVolume claim to store the backup of NeuVector config files, please see the general Backup/Persistent Data section in the [Deploying NeuVector overview](https://open-docs.neuvector.com/deploying/production/#backups-and-persistent-data). If your deployment supports an integrated load balancer, change type NodePort to LoadBalancer for the console in the yaml file below.

When deploying `SUSE NeuVector` using the kubernetes YAML file, there are several CRDs, Service Accounts, Cluster Roles, and Cluster Role Bindings that need to be deployed as well.

---

============================================================================

## Installing SUSE NeuVector Using Kubernetes YAML File - Step-By-Step Guide

============================================================================

This section is a ste-by-step guide for installing the `SUSE NeuVector` using Kubernetes YAML File.

1. Create the NeuVector namespace
```bash
kubectl create ns neuvector
```

2. create the required service accounts
```bash
kubectl create sa controller -n neuvector
kubectl create sa enforcer -n neuvector
kubectl create sa basic -n neuvector
kubectl create sa updater -n neuvector
kubectl create sa scanner -n neuvector
kubectl create sa registry-adapter -n neuvector
```

3. Create the custom resources (CRD) for NeuVector security rules - This is for kubernetes version 1.19 and above.
```bash
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/waf-crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/dlp-crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/com-crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/vul-crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/admission-crd-k8s-1.19.yaml
```

4. Create the ClusterRoles to apply the read permissions required.
```bash
kubectl create clusterrole neuvector-binding-app --verb=get,list,watch,update --resource=nodes,pods,services,namespaces
kubectl create clusterrole neuvector-binding-rbac --verb=get,list,watch --resource=rolebindings.rbac.authorization.k8s.io,roles.rbac.authorization.k8s.io,clusterrolebindings.rbac.authorization.k8s.io,clusterroles.rbac.authorization.k8s.io
kubectl create clusterrolebinding neuvector-binding-app --clusterrole=neuvector-binding-app --serviceaccount=neuvector:controller
kubectl create clusterrolebinding neuvector-binding-rbac --clusterrole=neuvector-binding-rbac --serviceaccount=neuvector:controller
kubectl create clusterrole neuvector-binding-admission --verb=get,list,watch,create,update,delete --resource=validatingwebhookconfigurations,mutatingwebhookconfigurations
kubectl create clusterrolebinding neuvector-binding-admission --clusterrole=neuvector-binding-admission --serviceaccount=neuvector:controller
kubectl create clusterrole neuvector-binding-customresourcedefinition --verb=watch,create,get,update --resource=customresourcedefinitions
kubectl create clusterrolebinding neuvector-binding-customresourcedefinition --clusterrole=neuvector-binding-customresourcedefinition --serviceaccount=neuvector:controller
kubectl create clusterrole neuvector-binding-nvsecurityrules --verb=get,list,delete --resource=nvsecurityrules,nvclustersecurityrules
kubectl create clusterrole neuvector-binding-nvadmissioncontrolsecurityrules --verb=get,list,delete --resource=nvadmissioncontrolsecurityrules
kubectl create clusterrole neuvector-binding-nvdlpsecurityrules --verb=get,list,delete --resource=nvdlpsecurityrules
kubectl create clusterrole neuvector-binding-nvwafsecurityrules --verb=get,list,delete --resource=nvwafsecurityrules
kubectl create clusterrolebinding neuvector-binding-nvsecurityrules --clusterrole=neuvector-binding-nvsecurityrules --serviceaccount=neuvector:controller
kubectl create clusterrolebinding neuvector-binding-view --clusterrole=view --serviceaccount=neuvector:controller
kubectl create clusterrolebinding neuvector-binding-nvwafsecurityrules --clusterrole=neuvector-binding-nvwafsecurityrules --serviceaccount=neuvector:controller
kubectl create clusterrolebinding neuvector-binding-nvadmissioncontrolsecurityrules --clusterrole=neuvector-binding-nvadmissioncontrolsecurityrules --serviceaccount=neuvector:controller
kubectl create clusterrolebinding neuvector-binding-nvdlpsecurityrules --clusterrole=neuvector-binding-nvdlpsecurityrules --serviceaccount=neuvector:controller
kubectl create role neuvector-binding-scanner --verb=get,patch,update,watch --resource=deployments -n neuvector
kubectl create rolebinding neuvector-binding-scanner --role=neuvector-binding-scanner --serviceaccount=neuvector:updater --serviceaccount=neuvector:controller -n neuvector
kubectl create role neuvector-binding-secret --verb=get --resource=secrets -n neuvector
kubectl create rolebinding neuvector-binding-secret --role=neuvector-binding-secret --serviceaccount=neuvector:controller -n neuvector
kubectl create clusterrole neuvector-binding-nvcomplianceprofiles --verb=get,list,delete --resource=nvcomplianceprofiles
kubectl create clusterrolebinding neuvector-binding-nvcomplianceprofiles --clusterrole=neuvector-binding-nvcomplianceprofiles --serviceaccount=neuvector:controller
kubectl create clusterrole neuvector-binding-nvvulnerabilityprofiles --verb=get,list,delete --resource=nvvulnerabilityprofiles
kubectl create clusterrolebinding neuvector-binding-nvvulnerabilityprofiles --clusterrole=neuvector-binding-nvvulnerabilityprofiles --serviceaccount=neuvector:controller
```

5. Create the NeuVector Services and Pods using the sample file available in this [link](https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.3.0/neuvector-k8s.yaml), or download and modify as required. In this example, we have downloaded it and modified several options as follow: (also available in the [YAML-File Directory of this repo](/1-Install/NeuVector/2-Install-Neuvector-With-Kubernetes-Yaml/YAML-Files/NueVector-Yaml-File.yaml))
- Changed the Manager service from LoadBalancer to NodePort
- Changed all PODs tags from 5.3.3 to 5.3.4 except scanner and updater - left as latest and they has to be set as latest for the updates.
```bash
mkdir neuvector-yaml && cd neuvector-yaml
vi neuvector-yaml-deploy.yaml
kubectl apply -f neuvector-yaml-deploy.yaml
```

6. Optionally, create an ingress to access the NeuVector. You can access the NeuVector using External LoadBalancer (configure the manager service as a LoadBalancer), NodePort (configure the manager service as a NodePort), or Ingress. In the [YAML-File Directory](/1-Install/NeuVector/2-Install-Neuvector-With-Kubernetes-Yaml/YAML-Files/NeuVector-Ingress.yaml) of this repo, there is a yaml file to deploy an ingress for NeuVector. Edit it as you see fit.
```bash
vi neuvector-ingress.yaml
kubectl apply -f neuvector-ingress.yaml
```

---

## References

- [SUSE NeuVector - Deploying NeuVector](https://open-docs.neuvector.com/deploying/production/#backups-and-persistent-data)
- [SUSE NeuVector - Deploying Using Kubernetes YAML](https://open-docs.neuvector.com/deploying/kubernetes)

---

**Enjoy** :blush:

