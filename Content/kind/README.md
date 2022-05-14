## Instalação e Configuração do Kind.

<p align="center">
  <img width="500" height="350" src="https://d33wubrfki0l68.cloudfront.net/d0c94836ab5b896f29728f3c4798054539303799/9f948/logo/logo.png">
</p>

O [**KinD**](https://kind.sigs.k8s.io/) (Kubernetes in Docker) é uma ferramenta para executar o Kubernetes em containers [**Docker**](https://docs.docker.com/). O Kind foi inclusive projetado para testar o próprio Kubernetes.

Como pré-requisito, você precisa ter o Docker devidamente instalado e funcional. [Clicando aqui](https://docs.docker.com/get-docker/) você será direcionado para a documentação de instalação do Docker.

**01. Download e instalação do kind.**

Na página do [**Github**](https://github.com/kubernetes-sigs/kind/releases) do projeto você encontra os arquivos compilados para diversas distribuições.

Exemplo de instalação em um GNU/Linux x86_64.

```bash
wget https://github.com/kubernetes-sigs/kind/releases/download/v0.13.0/kind-linux-amd64
chmod +x kind-linux-amd64
mv kind-linux-amd64 /usr/local/bin/kind
kind version
```

**02. Inicializando o cluster.**

É possível inicializar o kind com apenas um nó contendo todas as funções necessárias do Kubernetes.

Um exemplo prático disso é executando o comando abaixo que irá subir um único node.

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
    image: docker.io/kindest/node:v1.24.0@sha256:406fd86d48eaf4c04c7280cd1d2ca1d61e7d0d61ddef0125cb097bc7b82ed6a1
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
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP

  - role: worker
    image: docker.io/kindest/node:v1.24.0@sha256:406fd86d48eaf4c04c7280cd1d2ca1d61e7d0d61ddef0125cb097bc7b82ed6a1
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
    image: docker.io/kindest/node:v1.24.0@sha256:406fd86d48eaf4c04c7280cd1d2ca1d61e7d0d61ddef0125cb097bc7b82ed6a1
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
  apiServerAddress: "127.0.0.1"
  apiServerPort: 6443
  disableDefaultCNI: true
```

Observe que no YAML informei a versão `1.24.0` do Kubernetes. O kind na versão que estou utilizando, `v0.13.0`, suporta as versões abaixo do Kubernetes:
```
1.24: kindest/node:v1.24.0@sha256:406fd86d48eaf4c04c7280cd1d2ca1d61e7d0d61ddef0125cb097bc7b82ed6a1
1.23: kindest/node:v1.23.6@sha256:1af0f1bee4c3c0fe9b07de5e5d3fafeb2eec7b4e1b268ae89fcab96ec67e8355
1.22: kindest/node:v1.22.9@sha256:6e57a6b0c493c7d7183a1151acff0bfa44bf37eb668826bf00da5637c55b6d5e
1.21: kindest/node:v1.21.12@sha256:ae05d44cc636ee961068399ea5123ae421790f472c309900c151a44ee35c3e3e
1.20: kindest/node:v1.20.15@sha256:a6ce604504db064c5e25921c6c0fffea64507109a1f2a512b1b562ac37d652f3
1.19: kindest/node:v1.19.16@sha256:dec41184d10deca01a08ea548197b77dc99eeacb56ff3e371af3193c86ca99f4
1.18: kindest/node:v1.18.20@sha256:38a8726ece5d7867fb0ede63d718d27ce2d41af519ce68be5ae7fcca563537ed
```

Agora utililize o comando abaixo para inicializar o cluster.

```bash
kind create cluster --name kindcluster --config kind.yaml --kubeconfig ~/.kube/kind.yaml
```

Com o cluster inicializado, exporte a variável `KUBECONFIG`, pois no comando acima foi informado onde seria escrito o arquivo `kubeconfig file`.
```bash
export KUBECONFIG="~/.kube/kind.yaml"
```

**03. Instalação do CNI Weave Net.**

Executando o comando ```kubectl get nodes``` percebe-se que o status dos nodes será **NotReady**, pois o CNI padrão, o **kindnet**, está desabilitado. Dessa forma os pods não podem se comunicar.

Faço isso, porque prefiro estudar/trabalhar com o CNI [**Weave-net**](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/), que para instalar basta executar o comando abaixo.

```bash
kubectl apply --filename "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
Finalizado o deployment do `weave-net` você terá um cluster pronto para estudo.

**04. StorageClass padrão.**

Por padrão o `Kind` instala o [`local-path-provisioner`](https://github.com/rancher/local-path-provisioner) que nos permite trabalhar com persistência de dados.

Perceba que no arquivo `kind.yaml` utilizado para inicializar o cluster temos um mapeamento de diretório.

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