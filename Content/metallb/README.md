## Instalação e configuração do MetalLB

Com o [**MetalLB**](https://metallb.universe.tf/) podemos utilizar o recurso de [`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/ingress/#load-balancing) mesmo estando executando um cluster local ou em ambientes On-Premises.

Antes de implantar o MetalLB, precisamos definir um pool de IPs para alocação e uso dos serviços do tipo `LoadBalancer`.

Verifique qual é a rede do Docker que o `kind` está utilizando.
```bash
docker network inspect kind | jq '.[].IPAM | .Config | .[0].Subnet' | cut -d \" -f 2
```

No meu caso, o resultado foi `172.19.0.0/16`. Sabendo disso, irei alocar apenas um IP dessa faixa de rede para ser utilizado no `Ingress Controller`:
```
172.19.0.100/32
```
Em alguns cenários se faz necessário alocar um range de IPs e isso pode ser feito da seguinte forma:
```
172.19.0.100-172.19.0.105
```
Dessa forma tenho um range de 5 IPs para serem alocados para serviços do tipo `LoadBalancer`.

---

Implante o MetalLB `v0.13.7`:
```bash
kubectl create --filename https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```

Aguarde até que os pods `controller` e `speakers`, no namespace `metallb-system`, estejam prontos:
```bash
kubectl wait --namespace metallb-system --for=condition=ready pod --selector=app=metallb --timeout=90s
```

---

Com os pods prontos, conclua a configuração `layer2` definindo e anunciando o pool de IPs que serão utilizados pelo Load Balancer.

```yaml
cat <<'EOF' | kubectl create --filename -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: development
  namespace: metallb-system
spec:
  addresses:
  - 172.19.0.100/32

---

apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: empty
  namespace: metallb-system
spec:
  ipAddressPools:
  - development
EOF
```

Você pode conferir as configurações aplicadas da seguinte forma:

```bash
kubectl get IPAddressPool,L2Advertisement --namespace metallb-system
```

Resultado:
```NAME                                   AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
ipaddresspool.metallb.io/development   true          false             ["172.19.0.100/32"]

NAME                               IPADDRESSPOOLS    IPADDRESSPOOL SELECTORS   INTERFACES
l2advertisement.metallb.io/empty   ["development"]
```