## Instalação e configuração do Metrics Server

O [**Metrics Server**](https://github.com/kubernetes-sigs/metrics-server) é um recurso do Kubernetes na qual coleta métricas de recursos do [**kubelet**](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) e as expõem  no [**Kubernetes API server**](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) por meio da API Metrics.

Para instalar o Metrics Server podemos simplesmente aplicar o arquivo de manifesto ```components.yaml```, mas aqui no meu caso que estou utilizando o Kind terei problemas relacionados ao certificado CA que não é válido.

Abaixo um exemplo do erro que ocorre:
```
E0919 18:23:43.157897       1 manager.go:111] unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:estudo-control-plane: unable to fetch metrics from Kubelet estudo-control-plane (estudo-control-plane): Get https://estudo-control-plane:10250/stats/summary?only_cpu_and_memory=true: x509: certificate signed by unknown authority, unable to fully scrape metrics from source kubelet_summary:estudo-worker: unable to fetch metrics from Kubelet estudo-worker (estudo-worker): Get https://estudo-worker:10250/stats/summary?only_cpu_and_memory=true: x509: certificate signed by unknown authority, unable to fully scrape metrics from source kubelet_summary:estudo-worker2: unable to fetch metrics from Kubelet estudo-worker2 (estudo-worker2): Get https://estudo-worker2:10250/stats/summary?only_cpu_and_memory=true: x509: certificate signed by unknown authority]
```

Diante disso, baixe o arquivo ```components.yml```.
```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
```

Procure o trecho do ```Deployment``` do ```metrics-server``` e passe para o container o argumento ```--kubelet-insecure-tls```. Dessa forma não será consultado a CA dos certificados gerados para os serviços do Kubernetes.

O ```Deployment``` ficará da seguinte forma:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server/metrics-server:v0.3.7
        imagePullPolicy: IfNotPresent
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
        ports:
        - name: main-port
          containerPort: 4443
          protocol: TCP
        securityContext:
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
      nodeSelector:
        kubernetes.io/os: linux
```

Feito isso, só criar o recurso.
```bash
kubectl create -f components.yaml
```

Após alguns minutos será possível consultar as métrics de recursos do seu cluster. Abaixo dois exemplos simples disso.

```
➜ kubectl top nodes                                                                       
NAME                   CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
estudo-control-plane   127m         3%     595Mi           3%        
estudo-worker          36m          0%     236Mi           1%        
estudo-worker2         54m          1%     256Mi           1%        
```

```
➜ kubectl top pods --all-namespaces            
NAMESPACE            NAME                                           CPU(cores)   MEMORY(bytes)   
kube-system          coredns-66bff467f8-jhnr8                       5m           7Mi             
kube-system          coredns-66bff467f8-vdqqg                       4m           7Mi             
kube-system          etcd-estudo-control-plane                      25m          27Mi            
kube-system          kube-apiserver-estudo-control-plane            48m          245Mi           
kube-system          kube-controller-manager-estudo-control-plane   17m          35Mi            
kube-system          kube-proxy-hrmwm                               1m           11Mi            
kube-system          kube-proxy-m6pkk                               1m           10Mi            
kube-system          kube-proxy-rz85r                               1m           11Mi            
kube-system          kube-scheduler-estudo-control-plane            5m           13Mi            
kube-system          metrics-server-8499f4d68c-qcrhv                1m           11Mi            
kube-system          weave-net-84d54                                2m           43Mi            
kube-system          weave-net-8nzw4                                2m           49Mi            
kube-system          weave-net-lswbz                                3m           39Mi            
local-path-storage   local-path-provisioner-67795f75bd-zmzhh        7m           7Mi             
metallb-system       controller-57f648cb96-xsqzg                    1m           5Mi             
metallb-system       speaker-cgf5l                                  8m           7Mi             
metallb-system       speaker-k5q4d                                  5m           7Mi             
metallb-system       speaker-rdp5b                                  7m           7Mi             
```