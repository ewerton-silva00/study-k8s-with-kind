## Instalação e Configuração do Kind.

<p align="center"><img alt="kind" src="[https://raw.githubusercontent.com/kubernetes-sigs/kind/master/logo/logo.png)" width="300px" /></p>

O [**Kind**](https://kind.sigs.k8s.io/) (Kubernetes in Docker) é uma ferramenta para executar o Kubernetes local em containers [**Docker**](https://docs.docker.com/). O Kind foi inclusive projetado para testar o próprio Kubernetes.

Como pré-requisito, você precisa ter o Docker devidamente instalado e funcional. [Clicando aqui](https://docs.docker.com/get-docker/) você será direcionado para a documentação de instalação do Docker.

**01. Download e instalação do kind.**

Na página do [**Github**](https://github.com/kubernetes-sigs/kind/releases) do projeto você encontra os arquivos compilados para diversas distribuições.

Exemplo de instalação em um GNU/Linux x86_64.

```bash
wget https://github.com/kubernetes-sigs/kind/releases/download/v0.10.0/kind-linux-amd64
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

No meu caso, irei inicializar um cluster com 3 nodes, 1 **control-plane** e 2 **workers**, a partir de um arquivo de configuração YAML (YAML Engineer ❤️).

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: docker.io/kindest/node:v1.18.15@sha256:5c1b980c4d0e0e8e7eb9f36f7df525d079a96169c8a8f20d8bd108c0d0889cc4
    kubeadmConfigPatches:
    - |
      kind: InitConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "ingress-ready=true"
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP

  - role: worker
    image: docker.io/kindest/node:v1.18.15@sha256:5c1b980c4d0e0e8e7eb9f36f7df525d079a96169c8a8f20d8bd108c0d0889cc4
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "workloads=true"

  - role: worker
    image: docker.io/kindest/node:v1.18.15@sha256:5c1b980c4d0e0e8e7eb9f36f7df525d079a96169c8a8f20d8bd108c0d0889cc4
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "workloads=true"

networking:
  disableDefaultCNI: true
```

Observe que no YAML informei a versão `1.18.5` do Kubernetes. O kind na versão que estou utilizando, `v0.10.0`, suporta as versões abaixo do Kubernetes:
```
1.20: kindest/node:v1.20.2@sha256:8f7ea6e7642c0da54f04a7ee10431549c0257315b3a634f6ef2fecaaedb19bab
1.19: kindest/node:v1.19.7@sha256:a70639454e97a4b733f9d9b67e12c01f6b0297449d5b9cbbef87473458e26dca
1.18: kindest/node:v1.18.15@sha256:5c1b980c4d0e0e8e7eb9f36f7df525d079a96169c8a8f20d8bd108c0d0889cc4
1.17: kindest/node:v1.17.17@sha256:7b6369d27eee99c7a85c48ffd60e11412dc3f373658bc59b7f4d530b7056823e
1.16: kindest/node:v1.16.15@sha256:c10a63a5bda231c0a379bf91aebf8ad3c79146daca59db816fb963f731852a99
1.15: kindest/node:v1.15.12@sha256:67181f94f0b3072fb56509107b380e38c55e23bf60e6f052fbd8052d26052fb5
1.14: kindest/node:v1.14.10@sha256:3fbed72bcac108055e46e7b4091eb6858ad628ec51bf693c21f5ec34578f6180
```

Agora utililize o comando abaixo para inicializar o cluster.

```bash
kind create cluster --name kindcluster --config kind.yml --kubeconfig ~/.kube/kind.yml
```

Com o cluster inicializado, exporte a variável `KUBECONFIG`, pois no comando acima foi informado onde seria escrito o arquivo `kubeconfigfile`.
```bash
export KUBECONFIG="~/.kube/kind.yml"
```

Executando o comando ```kubectl get nodes``` percebe-se que o status dos nodes será **NotReady**, pois o CNI padrão, o **kindnet**, está desabilitado. Dessa forma os pods não podem se comunicar.

Faço isso, porque prefiro estudar/trabalhar com o CNI [**Weave-net**](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/), que para instalar basta executar o comando abaixo.

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
Finalizado o deployment do `weave-net` você terá um cluster pronto para estudo.