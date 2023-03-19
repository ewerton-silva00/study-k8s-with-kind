## Instalação e configuração do Metrics Server

O [**Metrics Server**](https://github.com/kubernetes-sigs/metrics-server) coleta métricas de recursos do [**kubelet**](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) e as expõem  no [**Kubernetes API server**](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) por meio da API Metrics.

Baixe a versão mais recente e estável do `metrics-server`.
```bash
curl --location https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.2/components.yaml --output metrics-server-v0.6.2.yaml
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
kubectl crate --filename metrics-server-v0.6.2.yaml
```

Aguarde o pod ficar pronto.
```bash
kubectl wait --namespace kube-system --for=condition=ready pod --selector=k8s-app=metrics-server --timeout=90s
```

Após alguns minutos será possível consultar as métrics de recursos do seu cluster. Abaixo dois exemplos simples disso.

```
➜ kubectl top nodes
NAME                        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
kindcluster-control-plane   154m         1%     740Mi           3%
kindcluster-worker          36m          0%     177Mi           0%
kindcluster-worker2         42m          0%     182Mi           0%
```

```
➜ kubectl top pods --all-namespaces
NAMESPACE            NAME                                                CPU(cores)   MEMORY(bytes)
kube-system          coredns-565d847f94-rhtp9                            3m           13Mi
kube-system          coredns-565d847f94-wq4zz                            2m           13Mi
kube-system          etcd-kindcluster-control-plane                      29m          42Mi
kube-system          kindnet-4lnp4                                       2m           8Mi
kube-system          kindnet-8rbz9                                       1m           8Mi
kube-system          kindnet-xxqkj                                       1m           8Mi
kube-system          kube-apiserver-kindcluster-control-plane            50m          291Mi
kube-system          kube-controller-manager-kindcluster-control-plane   18m          45Mi
kube-system          kube-proxy-5q5qj                                    1m           10Mi
kube-system          kube-proxy-9tqvc                                    1m           10Mi
kube-system          kube-proxy-rrbsh                                    1m           10Mi
kube-system          kube-scheduler-kindcluster-control-plane            4m           18Mi
kube-system          metrics-server-55dd79d7bf-9qswk                     4m           18Mi
local-path-storage   local-path-provisioner-684f458cdd-cc8cr             1m           7Mi
metallb-system       controller-84d6d4db45-6hh7w                         2m           17Mi
metallb-system       speaker-2z4lb                                       5m           15Mi
metallb-system       speaker-tpwnh                                       5m           17Mi
metallb-system       speaker-wm7lc                                       5m           17Mi
```
