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
wget https://github.com/kubernetes-sigs/kind/releases/download/v0.17.0/kind-linux-amd64
chmod +x kind-linux-amd64
mv kind-linux-amd64 /usr/local/bin/kind
kind version
```

**02. Inicializando o cluster.**

É possível inicializar o kind com apenas um nó contendo todas as funções necessárias do Kubernetes.

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
    image: docker.io/kindest/node:v1.25.3@sha256:f52781bc0d7a19fb6c405c2af83abfeb311f130707a0e219175677e366cc45d1
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
    image: docker.io/kindest/node:v1.25.3@sha256:f52781bc0d7a19fb6c405c2af83abfeb311f130707a0e219175677e366cc45d1
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
    image: docker.io/kindest/node:v1.25.3@sha256:f52781bc0d7a19fb6c405c2af83abfeb311f130707a0e219175677e366cc45d1
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

Observe que no YAML informei a versão `1.25.3` do Kubernetes. O kind na versão que estou utilizando, `v0.17.0`, suporta as versões abaixo do Kubernetes:
```
1.25: kindest/node:v1.25.3@sha256:f52781bc0d7a19fb6c405c2af83abfeb311f130707a0e219175677e366cc45d1
1.24: kindest/node:v1.24.7@sha256:577c630ce8e509131eab1aea12c022190978dd2f745aac5eb1fe65c0807eb315
1.23: kindest/node:v1.23.13@sha256:ef453bb7c79f0e3caba88d2067d4196f427794086a7d0df8df4f019d5e336b61
1.22: kindest/node:v1.22.15@sha256:7d9708c4b0873f0fe2e171e2b1b7f45ae89482617778c1c875f1053d4cef2e41
1.21: kindest/node:v1.21.14@sha256:9d9eb5fb26b4fbc0c6d95fa8c790414f9750dd583f5d7cee45d92e8c26670aa1
1.20: kindest/node:v1.20.15@sha256:a32bf55309294120616886b5338f95dd98a2f7231519c7dedcec32ba29699394
1.19: kindest/node:v1.19.16@sha256:476cb3269232888437b61deca013832fee41f9f074f9bed79f57e4280f7c48b7
```

Agora utililize o comando abaixo para inicializar o cluster.

```bash
kind create cluster --name kindcluster --config kind.yaml --kubeconfig ~/.kube/kind.yaml
```

Com o cluster inicializado, exporte a variável `KUBECONFIG`, pois no comando acima foi informado onde seria escrito o arquivo `kubeconfig file`.
```bash
export KUBECONFIG="~/.kube/kind.yaml"
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