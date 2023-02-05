## Instalação e configuração do MetalLB

Com o [**MetalLB**](https://metallb.universe.tf/) podemos utilizar o recurso de [`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/ingress/#load-balancing) mesmo estando executando um cluster local ou em ambientes On-Premises.

Antes de implantar o MetalLB, precisamos definir um pool de IPs para alocação e uso dos serviços do tipo `LoadBalancer`.

Verifique qual é a rede que o `kind` está utilizando.
```bash
docker network inspect kind | jq '.[].IPAM | .Config | .[0].Subnet' | cut -d \" -f 2
```

No meu caso, o resultado foi `172.19.0.0/16`. Sabendo disso, irei alocar `5` IPs dessa faixa de rede, que serão:
```
172.19.0.10-172.19.0.15
```
> Em cenários de testes locais alocar um range de IPs pode ser não necessário. Você pode alocar apenas um IP para ser utilizado pelo Ingress Controller, por exemplo.

Implante o MetalLB `v0.13.7`:
```bash
kubectl create --filename https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```

Aguarde até que os pods `controller` e `speakers`, no namespace `metallb-system`, estejam prontos:
```bash
kubectl wait --namespace metallb-system --for=condition=ready pod --selector=app=metallb --timeout=90s
```

Com os pods prontos, conclua a configuração `layer2` definindo e anunciando o pool de IPs que serão utilizados pelo Load Balancer.
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: development
  namespace: metallb-system
spec:
  addresses:
  - 172.19.0.10-172.19.0.15

---

apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
spec:
  ipAddressPools:
  - development
```

```bash
kubectl create --filename metallb-config.yaml
```
