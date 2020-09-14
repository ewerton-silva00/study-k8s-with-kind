## Estudo de Kubernetes com Kind

O [**Kind**](https://kind.sigs.k8s.io/) (Kubernetes in Docker) é uma forma fácil e leve de executar o Kubernetes num ambiente local para testes e aprendizado, mas não é recomendado para uso em produção. O Kind foi inclusive desenvolvido para testar o próprio Kubernetes.

**01. Download e instalação do kind.**

Na página do [**Github**](https://github.com/kubernetes-sigs/kind/releases) do projeto você encontra os arquivos compilados para diversas distribuições.

Exemplo de instalação em um GNU/Linux x86_64.

```bash
wget https://github.com/kubernetes-sigs/kind/releases/download/v0.8.1/kind-linux-amd64
chmod +x kind-linux-amd64
mv kind-linux-amd64 /usr/local/bin/kind
kind version
```
> Importante entender que para executar o kind se faz necessário ter o Docker devidamente instalado e funcional. Para saber mais como instalar e configurar o Docker consulte a [**documentação oficial**](https://docs.docker.com/engine/install/).

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
    image: docker.io/kindest/node:v1.18.8
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
    image: docker.io/kindest/node:v1.18.8
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "workloads=true"

  - role: worker
    image: docker.io/kindest/node:v1.18.8
    kubeadmConfigPatches:
    - |
      kind: JoinConfiguration
      nodeRegistration:
        kubeletExtraArgs:
          node-labels: "workloads=true"

networking:
  disableDefaultCNI: true
```

Agora utilizo o comando abaixo para inicializar o cluster.

```bash
kind create cluster --name estudo --config kind.yaml
```

Com o cluster inicializado, executando o comando ```kubectl get nodes``` percebe-se que o status dos nodes será **NotReady**, pois desabilitei o CNI padrão, o **kindnet**. Dessa forma os pods não podem se comunicar.

Faço isso, porque prefiro estudar/trabalhar com o CNI [**Weave-net**](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/), que para instalar executo o comando abaixo.

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
Finalizado o deployment do weave-net tenho um cluster pronto para estudo.