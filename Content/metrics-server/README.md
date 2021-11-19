## Instalação e configuração do Metrics Server

O [**Metrics Server**](https://github.com/kubernetes-sigs/metrics-server) coleta métricas de recursos do [**kubelet**](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) e as expõem  no [**Kubernetes API server**](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) por meio da API Metrics.

Utilizaremos o [`helm`](https://helm.sh/) para instalar esse componente. Coisa simples.

Instale o repositório do Helm Chart:
```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
```

Agora, com um único comando instale o `Metrics Server`.
```bash
helm install \
metrics-server metrics-server/metrics-server \
  --version 3.7.0 \
  --namespace kube-system \
  --set rbac.create=true \
  --set args={"--kubelet-insecure-tls"} \
  --set apiService.create=true
kubectl wait --for condition=Available=True deploy/metrics-server --namespace kube-system --timeout -1s
```

Após alguns minutos será possível consultar as métrics de recursos do seu cluster. Abaixo dois exemplos simples disso.

```
➜ kubectl top nodes                                                                       
NAME                   CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
dev-control-plane   127m         3%     595Mi           3%        
dev-worker          36m          0%     236Mi           1%        
dev-worker2         54m          1%     256Mi           1%        
```

```
➜ kubectl top pods --all-namespaces            
NAMESPACE            NAME                                        CPU(cores)   MEMORY(bytes)   
kube-system          coredns-74ff55c5b-dvgmz                     4m           9Mi             
kube-system          coredns-74ff55c5b-pf49w                     3m           9Mi             
kube-system          etcd-dev-control-plane                      14m          50Mi            
kube-system          kube-apiserver-dev-control-plane            62m          305Mi           
kube-system          kube-controller-manager-dev-control-plane   12m          66Mi            
kube-system          kube-proxy-7nmj5                            1m           21Mi            
kube-system          kube-proxy-b9gz8                            1m           22Mi            
kube-system          kube-proxy-pm97w                            1m           18Mi            
kube-system          kube-scheduler-dev-control-plane            3m           27Mi            
kube-system          metrics-server-687b57545b-bb757             2m           16Mi            
kube-system          weave-net-757cq                             1m           42Mi            
kube-system          weave-net-7hrlw                             1m           45Mi            
kube-system          weave-net-t9g7m                             1m           39Mi            
local-path-storage   local-path-provisioner-547f784dff-tmq66     2m           7Mi
```