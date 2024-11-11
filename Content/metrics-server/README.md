## Instalação e configuração do Metrics Server

O [Metrics Server](https://github.com/kubernetes-sigs/metrics-server) coleta métricas de recursos do [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) e as expõem  no [Kubernetes API server](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) por meio da API [Metrics](https://github.com/kubernetes/metrics).

Baixe a versão mais recente e estável do `metrics-server`.

```bash
curl --location https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.2/components.yaml --output metrics-server-v0.7.2.yaml
```

Adicione a flag `--kubelet-insecure-tls` como argumento no container do metrics-server.

```yaml
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: k8s.gcr.io/metrics-server/metrics-server:v0.6.2
```

Implante o metrics-server:

```bash
kubectl create --filename metrics-server-v0.7.2.yaml
```

Aguarde o pod ficar pronto.

```bash
kubectl wait --namespace kube-system --for=condition=ready pod --selector=k8s-app=metrics-server --timeout=90s
```

Após alguns minutos será possível consultar as métrics de recursos do seu cluster. Abaixo dois exemplos simples disso.

```
➜ kubectl top nodes
NAME                CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
lab-control-plane   133m         1%     579Mi           2%        
lab-worker          32m          0%     181Mi           0%        
lab-worker2         28m          0%     176Mi           0%
```

```
➜ kubectl top pods --all-namespaces
NAMESPACE            NAME                                        CPU(cores)   MEMORY(bytes)   
kube-system          coredns-7c65d6cfc9-45xfp                    2m           14Mi            
kube-system          coredns-7c65d6cfc9-sft47                    3m           14Mi            
kube-system          etcd-lab-control-plane                      23m          42Mi            
kube-system          kindnet-gdv5b                               1m           11Mi            
kube-system          kindnet-kmbq5                               1m           10Mi            
kube-system          kindnet-rbl4c                               1m           10Mi            
kube-system          kube-apiserver-lab-control-plane            48m          209Mi           
kube-system          kube-controller-manager-lab-control-plane   23m          47Mi            
kube-system          kube-proxy-mtvxp                            7m           14Mi            
kube-system          kube-proxy-pt7qm                            5m           14Mi            
kube-system          kube-proxy-qntvd                            8m           14Mi            
kube-system          kube-scheduler-lab-control-plane            4m           17Mi            
kube-system          metrics-server-587b667b55-sxxkh             3m           19Mi            
local-path-storage   local-path-provisioner-57c5987fd4-m6qgz     1m           7Mi             
metallb-system       controller-8694df9d9b-lkbjg                 2m           16Mi            
metallb-system       speaker-6lg9g                               4m           16Mi            
metallb-system       speaker-q42xs                               8m           16Mi            
metallb-system       speaker-tllsr                               6m           16Mi
```
