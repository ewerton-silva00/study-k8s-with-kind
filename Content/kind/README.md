## Instalação e Configuração do Kind.

<p align="center">
  <img width="500" height="350" src="https://d33wubrfki0l68.cloudfront.net/d0c94836ab5b896f29728f3c4798054539303799/9f948/logo/logo.png">
</p>

O [**KinD**](https://kind.sigs.k8s.io/) (Kubernetes in Docker) é uma ferramenta para executar o Kubernetes em containers [**Docker**](https://docs.docker.com/). O Kind foi inclusive projetado para testar o próprio Kubernetes.

Como pré-requisitos, você precisa ter o Docker e o kubectl devidamente instalados e funcionais. [Clicando aqui](https://docs.docker.com/get-docker/) você será direcionado para a documentação de instalação do Docker.

**00. Download e instalação do kubectl.**

O [**kubectl**](https://kubernetes.io/docs/reference/kubectl/kubectl/) é a ferramenta de linha de comando que você irá usar para administrar seus clusters Kubernetes. A [documentação oficial](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) sugere várias maneiras de instalá-lo. Se você estiver usando Ubuntu Linux, use o repositório de pacotes binários seguindo os passos a seguir, porque será mais fácil mantê-lo autalizado no sistema:

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
kubectl version
```

**01. Download e instalação do kind.**

Na página do [**Github**](https://github.com/kubernetes-sigs/kind/releases) do projeto você encontra os arquivos compilados para diversas distribuições.

Exemplo de instalação em um GNU/Linux x86_64.

```bash
wget https://github.com/kubernetes-sigs/kind/releases/download/v0.11.1/kind-linux-amd64
sudo install -o root -g root -m 0755 kind-linux-amd64 /usr/local/bin/kind
kind version
```

**02. Inicializando o cluster.**

É possível inicializar o kind com apenas um nó contendo todas as funções necessárias do Kubernetes.

Um exemplo prático disso é executando o comando abaixo que irá criar um cluster com um único node.

```bash
kind create cluster
```

Isso irá criar um cluster chamado **kind** com apenas um nó.

> Execute ```kind create cluster --help``` para entender todas as possibilidades de uso.

Para listar os clusters e nós existentes, execute:

```bash
kind get clusters
kind get nodes
```

Para destruir o cluster e seus nós, execute:

```bash
kind delete cluster kind
```

Para um exemplo mais realista, vamos criar um cluster com 3 nodes, 1 **control-plane** e 2 **workers**, a partir de um arquivo de configuração YAML (YAML Engineer ❤️).

Crie um arquivo chamado ```kind.yaml``` no diretório corrente com o seguinte conteúdo:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: docker.io/kindest/node:v1.20.7@sha256:cbeaf907fc78ac97ce7b625e4bf0de16e3ea725daf6b04f930bd14c67c671ff9
    kubeadmConfigPatches:
    - |
      kind: InitConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "loadbalancer=enabled"
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
    image: docker.io/kindest/node:v1.20.7@sha256:cbeaf907fc78ac97ce7b625e4bf0de16e3ea725daf6b04f930bd14c67c671ff9
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "workloads=enabled"
    extraMounts:
      - hostPath: /tmp/local-path-provisioner
        containerPath: /var/local-path-provisioner

  - role: worker
    image: docker.io/kindest/node:v1.20.7@sha256:cbeaf907fc78ac97ce7b625e4bf0de16e3ea725daf6b04f930bd14c67c671ff9
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "workloads=enabled"
    extraMounts:
      - hostPath: /tmp/local-path-provisioner
        containerPath: /var/local-path-provisioner

networking:
  apiServerPort: 6443
  disableDefaultCNI: true
```

Observe que no YAML informei a versão `1.20.7` do Kubernetes. O kind na versão que estou utilizando, `v0.11.1`, suporta as versões abaixo do Kubernetes:
```
1.21: kindest/node:v1.21.1@sha256:69860bda5563ac81e3c0057d654b5253219618a22ec3a346306239bba8cfa1a6
1.20: kindest/node:v1.20.7@sha256:cbeaf907fc78ac97ce7b625e4bf0de16e3ea725daf6b04f930bd14c67c671ff9
1.19: kindest/node:v1.19.11@sha256:07db187ae84b4b7de440a73886f008cf903fcf5764ba8106a9fd5243d6f32729
1.18: kindest/node:v1.18.19@sha256:7af1492e19b3192a79f606e43c35fb741e520d195f96399284515f077b3b622c
1.17: kindest/node:v1.17.17@sha256:66f1d0d91a88b8a001811e2f1054af60eef3b669a9a74f9b6db871f2f1eeed00
1.16: kindest/node:v1.16.15@sha256:83067ed51bf2a3395b24687094e283a7c7c865ccc12a8b1d7aa673ba0c5e8861
1.15: kindest/node:v1.15.12@sha256:b920920e1eda689d9936dfcf7332701e80be12566999152626b2c9d730397a95
1.14: kindest/node:v1.14.10@sha256:f8a66ef82822ab4f7569e91a5bccaf27bceee135c1457c512e54de8c6f7219f8
```

Agora utililize o comando abaixo para criar o cluster.

```bash
kind create cluster --name kindcluster --config kind.yaml --kubeconfig ~/.kube/kind.yaml
```

Com o cluster inicializado, exporte a variável `KUBECONFIG`, pois no comando acima foi informado onde seria escrito o arquivo `kubeconfig file`.

```bash
export KUBECONFIG=~/.kube/kind.yaml
```

**03. Instalação do CNI Weave Net.**

Executando o comando ```kubectl get nodes``` percebe-se que o status dos nodes será **NotReady**, pois o CNI padrão, o **kindnet**, está desabilitado. Dessa forma os pods não podem se comunicar.

Faço isso, porque prefiro estudar/trabalhar com o CNI [**Weave-net**](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/), que para instalar basta executar o comando abaixo.

```bash
kubectl apply --filename "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 -w0)"
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
