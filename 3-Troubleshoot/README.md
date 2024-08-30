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



sudo mkdir -p /etc/rancher/rke2/
sudo cat << EOF >> /etc/rancher/rke2/config.yaml
write-kubeconfig-mode: "0644"
cni: "none"
disable-kube-proxy: true
token: “Suse!sTh3BestSince@1992”
EOF


sudo mkdir -p /etc/rancher/rke2/
sudo cat << EOF >> /etc/rancher/rke2/config.yaml
server: https://10.10.10.130:9345
token: “Suse!sTh3BestSince@1992”
EOF

helm repo add cilium https://helm.cilium.io
helm repo update
helm install cilium cilium/cilium --version 1.16.1 \
   --namespace kube-system \
   --set egressGateway.enabled=true \
   --set bpf.masquerade=true \
   --set kubeProxyReplacement=true
   --set k8sServiceHost=10.10.10.130
   --set k8sServicePort=6443

apiVersion: cilium.io/v2
kind: CiliumEgressGatewayPolicy
metadata:
  name: suse-poc-egress-gw-01
spec:
  selectors:
  - podSelector:
      matchLabels:
        io.kubernetes.pod.namespace: poc-egress-gw-01
  destinationCIDRs:
  - "0.0.0.0/0"
  egressGateway:
    nodeSelector:
      matchLabels:
        kubernetes.io/hostname: suse-poc-03-worker-01
    interface: eth0


apiVersion: v1
kind: Pod
metadata:
  name: mediabot-02
  namespace: poc-egress-gw-01
spec:
  containers:
  - name: mediabot
    image: quay.io/cilium/json-mock:v1.3.8@sha256:5aad04835eda9025fe4561ad31be77fd55309af8158ca8663a72f6abb78c2603
  nodeSelector:
    kubernetes.io/hostname: suse-poc-03-worker-02

