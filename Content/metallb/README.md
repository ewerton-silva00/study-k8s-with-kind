## Instalação e configuração do MetalLB

Com o [**MetalLB**](https://metallb.universe.tf/) podemos utilizar o recurso de LoadBalancer mesmo estando executando um cluster local.

Para instalar o MetalLB precisaremos do [**Helm**](https://helm.sh/), o gerenciador de pacotes do Kuberntes. Há [várias formas de instalá-lo](https://helm.sh/docs/intro/install/). Para sistemas baseados no Apt (como o Debian e o Ubuntu), é conveniente usar o repositório Apt mantido pelo projeto, pois assim será mais fácil mantê-lo atualizado:

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

Antes de instalar o MetalLB, precisamos definir um pool de IPs para alocação e uso do serviços do tipo `LoadBalancer`.

Verifique qual a rede que o `kind` está utilizando.
```bash
docker network inspect kind | jq -r '.[].IPAM.Config[0].Subnet'
```

No meu caso, o resultado foi `172.18.0.0/16`. Sabendo disso, irei alocar `10` IPs dessa faixa de rede, que serão:
```
172.18.0.10-172.18.0.20
```

Instale o repositório contendo o Helm Chart do MetalLB.
```bash
helm repo add metallb https://metallb.github.io/metallb
```

Instale o MetalLB.
```bash
helm upgrade --install metallb metallb/metallb \
  --version 0.11.0 \
  --create-namespace \
  --namespace metallb-system \
  --set "configInline.address-pools[0].addresses[0]="172.18.0.10-172.18.0.20"" \
  --set "configInline.address-pools[0].name=default" \
  --set "configInline.address-pools[0].protocol=layer2" \
  --set controller.nodeSelector.loadbalancer=enabled \
  --set "controller.tolerations[0].key=node-role.kubernetes.io/master" \
  --set "controller.tolerations[0].effect=NoSchedule" \
  --set speaker.tolerateMaster=true \
  --set speaker.nodeSelector.loadbalancer=enabled
kubectl wait --for condition=Available=True deploy/metallb-controller --namespace metallb-system --timeout -1s
kubectl wait --for condition=ready pod --selector app.kubernetes.io/component=controller --namespace metallb-system --timeout -1s
```
