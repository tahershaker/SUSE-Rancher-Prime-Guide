# SUSE NeuVector Configuration - Multi-Cluster Federation

This repo is created to provide the reader all the required information on configuring `SUSE NeuVector`. This repo provide a high-level explanation along with configure step-by-step guides and scripts.

---

<p align="center">
    <img src="Images/Rancher-Logo.png">
</p>

---

## About This Repo

This repo is designed and created to provide a documented step-by-step guide for configuring Multi-Cluster Federation for `SUSE NeuVector`. 

`SUSE NeuVector` is deployed per cluster (for each cluster `SUSE NeuVector` will be deployed in this cluster and will have it own management UI interface), however, for multi-cluster deployment, `SUSE NeuVector` provide the capability of federation to connect multiple `SUSE NeuVector` together and have a single pane of glass for managing these clusters together as well as the capability to create federated rules to be pushed on all (or some) clusters.

The NeuVector console can be used to manage large enterprise multi-cluster and multi-cloud deployments. One cluster should be selected as the Primary cluster, and other Remote clusters will then be able to join the Primary. Once connected, the Primary cluster can push Federated rules down to each remote cluster, which display as Federated rules in the consoles of each remote cluster. Scanned Federated registries will also sync the scan results with remote clusters. Only local users & Rancher users who have admin permission can promote a cluster to become the primary cluster. 

For more information regarding `SUSE NeuVector` multi-cluster management & federation, please refer to this [link](https://open-docs.neuvector.com/navigation/multicluster/)

---

==================================================================

## Step-By-Step Guide - Importing Existing Cluster To SUSE Rancher

==================================================================



---

## References

- [SUSE NeuVector Enterprise Multi-Cluster Management](https://open-docs.neuvector.com/navigation/multicluster/)

---

**Enjoy** :blush: