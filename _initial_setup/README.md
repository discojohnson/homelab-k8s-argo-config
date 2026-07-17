1. Create the cluster. The cluster is inatlled withoud CNI and kube-proxy

```bash
omnictl cluster template sync -f cluster-template/whatever.yaml
```

2. Install Cilium using Helm

```bash
CILIUM_VERSION=`curl -s https://raw.githubusercontent.com/cilium/cilium/refs/heads/main/stable.txt | sed 's/v//'`
helm repo add cilium https://helm.cilium.io/
helm repo update
helm install \
    cilium \
    cilium/cilium \
    --version ${CILIUM_VERSION} \
    --namespace kube-system \
    --set cgroup.autoMount.enabled=false \
    --set cgroup.hostRoot=/sys/fs/cgroup \
    --set=gatewayAPI.enabled=true \
    --set=gatewayAPI.enableAlpn=true \
    --set=gatewayAPI.enableAppProtocol=true \
    --set ipam.mode=kubernetes \
    --set k8sServiceHost=localhost \
    --set k8sServicePort=7445 \
    --set kubeProxyReplacement=true \
    --set securityContext.capabilities.ciliumAgent="{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}" \
    --set securityContext.capabilities.cleanCiliumState="{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}"
cilium status --wait
```

3. Install Argo CD with Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd --create-namespace
```

4. Seed Argo

```bash
kubectl create -f /homelab-k8s-argo-config/_initial_setup/argo-config.yaml
```
