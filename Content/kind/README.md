## Instalação e Configuração do Kind.

<p align="center">
  <img width="500" height="350" src="https://d33wubrfki0l68.cloudfront.net/d0c94836ab5b896f29728f3c4798054539303799/9f948/logo/logo.png">
</p>

O [**KinD**](https://kind.sigs.k8s.io/) (Kubernetes in Docker) é uma ferramenta para executar o Kubernetes em containers [**Docker**](https://docs.docker.com/).

Como pré-requisito, você precisa ter o [`Docker Engine`](https://docs.docker.com/engine/install/), [`kubectl`](https://kubernetes.io/docs/tasks/tools/#kubectl) e o [`Helm`](https://helm.sh/docs/intro/install/) devidamente instalados e funcionais.

**01. Download e instalação do kind.**

Na página do [**Github**](https://github.com/kubernetes-sigs/kind/releases) do projeto você encontra os arquivos compilados para diversas distribuições.

Exemplo de instalação em uma distribuição GNU/Linux x86_64.

```bash
curl --location --show-error --silent --output kind-linux-amd64 https://github.com/kubernetes-sigs/kind/releases/download/v0.25.0/kind-linux-amd64 && echo "b22ff7e5c02b8011e82cc3223d069d178b9e1543f1deb21e936d11764780a3d8 kind-linux-amd64" | sha256sum --check
sudo install --owner root --group root --mode 0755 kind-linux-amd64 /usr/local/bin/kind
rm -f kind-linux-amd64
kind version
```

**02. Inicializando o cluster.**

É possível inicializar o kind com apenas um nó contendo todos os componentes necessários do Kubernetes.

Um exemplo prático disso é executar o comando abaixo que irá subir um único node.

```bash
kind create cluster
```
> Utilize o **help** do kind para entender todas as possibilidades de uso.

No exemplo abaixo, temos um cluster com 3 nodes, 1 **control-plane** e 2 **workers**, a partir de um arquivo de configuração YAML (YAML Engineer ❤️).

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: docker.io/kindest/node:v1.31.2@sha256:18fbefc20a7113353c7b75b5c869d7145a6abd6269154825872dc59c1329912e
    kubeadmConfigPatches:
    - |
      kind: InitConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "node.kubernetes.io/loadbalancer=enabled"
    extraMounts:
      - hostPath: /tmp/local-path-provisioner
        containerPath: /var/local-path-provisioner
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        listenAddress: 0.0.0.0
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        listenAddress: 0.0.0.0
        protocol: TCP

  - role: worker
    image: docker.io/kindest/node:v1.31.2@sha256:18fbefc20a7113353c7b75b5c869d7145a6abd6269154825872dc59c1329912e
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "node.kubernetes.io/workloads=enabled"
    extraMounts:
      - hostPath: /tmp/local-path-provisioner
        containerPath: /var/local-path-provisioner

  - role: worker
    image: docker.io/kindest/node:v1.31.2@sha256:18fbefc20a7113353c7b75b5c869d7145a6abd6269154825872dc59c1329912e
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "node.kubernetes.io/workloads=enabled"
    extraMounts:
      - hostPath: /tmp/local-path-provisioner
        containerPath: /var/local-path-provisioner

networking:
  apiServerAddress: 127.0.0.1
  apiServerPort: 6443
```

Observe que no YAML informei a versão `1.31.2` do Kubernetes.
> [`Clique aqui`](https://kubernetes.io/releases/) para conferir todas as versões disponíveis do Kubernetes.

O kind na versão que estou utilizando, `v0.25.0`, suporta as versões abaixo do Kubernetes:

```
v1.31.0: kindest/node:v1.31.2@sha256:18fbefc20a7113353c7b75b5c869d7145a6abd6269154825872dc59c1329912e
v1.30.6: kindest/node:v1.30.6@sha256:b6d08db72079ba5ae1f4a88a09025c0a904af3b52387643c285442afb05ab994
v1.29.10: kindest/node:v1.29.10@sha256:3b2d8c31753e6c8069d4fc4517264cd20e86fd36220671fb7d0a5855103aa84b
v1.28.15: kindest/node:v1.28.15@sha256:a7c05c7ae043a0b8c818f5a06188bc2c4098f6cb59ca7d1856df00375d839251
v1.27.16: kindest/node:v1.27.16@sha256:2d21a61643eafc439905e18705b8186f3296384750a835ad7a005dceb9546d20
v1.26.15: kindest/node:v1.26.15@sha256:c79602a44b4056d7e48dc20f7504350f1e87530fe953428b792def00bc1076dd
```
> [`Clique aqui`](https://github.com/kubernetes-sigs/kind/releases) para conferir todas as versões disponíveis do Kind.

Utililize o comando abaixo para inicializar o cluster.
```bash
kind create cluster --name lab --config kind.yaml
```

Verifique se ficou tudo certo executando o comando abaixo:

```
➜ kubectl cluster-info --context kind-lab
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Verifique também se todos os pods estão em execução:

```
➜ kubectl get pods --all-namespaces --context kind-lab
NAMESPACE            NAME                                        READY   STATUS    RESTARTS   AGE
kube-system          coredns-7c65d6cfc9-45xfp                    1/1     Running   0          5m17s
kube-system          coredns-7c65d6cfc9-sft47                    1/1     Running   0          5m17s
kube-system          etcd-lab-control-plane                      1/1     Running   0          5m25s
kube-system          kindnet-gdv5b                               1/1     Running   0          5m13s
kube-system          kindnet-kmbq5                               1/1     Running   0          5m13s
kube-system          kindnet-rbl4c                               1/1     Running   0          5m17s
kube-system          kube-apiserver-lab-control-plane            1/1     Running   0          5m24s
kube-system          kube-controller-manager-lab-control-plane   1/1     Running   0          5m24s
kube-system          kube-proxy-mtvxp                            1/1     Running   0          5m13s
kube-system          kube-proxy-pt7qm                            1/1     Running   0          5m17s
kube-system          kube-proxy-qntvd                            1/1     Running   0          5m13s
kube-system          kube-scheduler-lab-control-plane            1/1     Running   0          5m24s
local-path-storage   local-path-provisioner-57c5987fd4-m6qgz     1/1     Running   0          5m17s
```

**04. StorageClass padrão.**

Por padrão o `Kind` instala o [`local-path-provisioner`](https://github.com/rancher/local-path-provisioner) que nos permite trabalhar com persistência de dados.

Perceba que no arquivo [`kind.yaml`](kind.yaml) utilizado para inicializar o cluster temos um mapeamento de diretório.

```
extraMounts:
  - hostPath: /tmp/local-path-provisioner
    containerPath: /var/local-path-provisioner
```

Ou seja, mapeamos o diretório `/tmp/local-path-provisioner` para o `/var/local-path-provisioner` dos containers do `kind`. Dessa forma temos um provisionamento dinâmico dos volumes persistentes.

Abaixo um exemplo de como ficam organizados os `PersistentVolumeClaim`.
```
/tmp/local-path-provisioner
├── pvc-499ceec2-1289-48c2-9f1f-b2224f1aba24_rocketchat_rocketchat-rocketchat
└── pvc-95410c9c-4391-4101-98df-e61709b70cb3_rocketchat_datadir-rocketchat-mongodb-0
    └── data
        └── db
            ├── diagnostic.data
            └── journal

6 directories
```

O `StorageClass` criado se chama `standard`, ou seja, no `storageClassName` é preciso passar o valor `standard`.

```
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  149m
```
