# SUSE Troubleshooting










Useful Commands

journalctl -u rke2-agent -f
sudo journalctl -u containerd
sudo systemctl restart containerd

/var/lib/rancher/rke2/server/db/etcd

df -h /var


kubectl get clusters.management.cattle.io  # find the cluster you want to delete 
export CLUSTERID="c-xxxxxxxxx" # 
kubectl patch clusters.management.cattle.io $CLUSTERID -p '{"metadata":{"finalizers":[]}}' --type=merge
kubectl delete clusters.management.cattle.io $CLUSTERID


kubectl describe node <node-name> | grep Taints
kubectl taint nodes <node-name> <taint-key>:<effect>-

/var/lib/rancher/rke2/bin/ctr --address /run/k3s/containerd/containerd.sock --namespace k8s.io container ls


export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
/var/lib/rancher/rke2/bin/crictl ps

/var/lib/rancher/rke2/bin/kubectl