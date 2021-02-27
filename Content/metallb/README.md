## Instalação e configuração do MetalLB

Com o [**MetalLB**](https://metallb.universe.tf/) consigo utilizar o recurso de LoadBalancer mesmo estando rodando o ambiente localmente, no meu computador.

Criação do namespace chamado ```metallb-system```.
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/namespace.yaml
```

Implantação do deployment contendo o ```controller``` e o daemonset contendo os ```speakers```.
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/metallb.yaml
```

Agora, a criação de uma ```secret``` contendo a ```secretKey``` que irá criptografar a comunicação entre os speakers.
```bash
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

Para finalizar, preciso apenas aplicar um ```ConfigMap``` contendo a configuração do MetalLB desejada, que nesse caso será do tipo ```Layer 2``` contendo um range de IPs que serão entregues aos serviços de LoadBalancer.

Para saber qual a subnet utilizada pelo Kind, basta consultar as configurações da bridge utilizada pelo Docker.
```bash
docker network inspect bridge | jq .
```

O resultado aqui no meu caso é o seguinte:
```
[
  {
    "Name": "bridge",
    "Id": "8702d3b4202ff0b322531ee480dd943aa2b68fc53c5fecb421d9a3fb2eb522da",
    "Created": "2020-09-13T20:13:00.486441023-03:00",
    "Scope": "local",
    "Driver": "bridge",
    "EnableIPv6": false,
    "IPAM": {
      "Driver": "default",
      "Options": null,
      "Config": [
        {
          "Subnet": "172.17.0.0/16",
          "Gateway": "172.17.0.1"
        }
      ]
    },
    "Internal": false,
    "Attachable": false,
    "Ingress": false,
    "ConfigFrom": {
      "Network": ""
    },
    "ConfigOnly": false,
    "Containers": {},
    "Options": {
      "com.docker.network.bridge.default_bridge": "true",
      "com.docker.network.bridge.enable_icc": "true",
      "com.docker.network.bridge.enable_ip_masquerade": "true",
      "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
      "com.docker.network.bridge.name": "docker0",
      "com.docker.network.driver.mtu": "1500"
    },
    "Labels": {}
  }
]
```

Ou seja, a subnet utilizada pelo Docker é ```172.17.0.0/16```. Diante disso, executando o comando ```kubectl get nodes -o wide``` posso ver os IPs atribuídos aos containers do Kind observando a coluna **INTERNAL-IP**.
```
NAME                   STATUS   ROLES    AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE           KERNEL-VERSION          CONTAINER-RUNTIME
estudo-control-plane   Ready    master   113m   v1.18.6   172.19.0.4    <none>        Ubuntu 20.04 LTS   5.8.4-200.fc32.x86_64   containerd://1.4.0-beta.2-21-ge818fe27
estudo-worker          Ready    <none>   113m   v1.18.6   172.19.0.3    <none>        Ubuntu 20.04 LTS   5.8.4-200.fc32.x86_64   containerd://1.4.0-beta.2-21-ge818fe27
estudo-worker2         Ready    <none>   113m   v1.18.6   172.19.0.2    <none>        Ubuntu 20.04 LTS   5.8.4-200.fc32.x86_64   containerd://1.4.0-beta.2-21-ge818fe27
```

Consigo inclusive ```pingar``` em cada um dos IPs listados.
```
➜ ping -c 1 172.19.0.4    
PING 172.19.0.4 (172.19.0.4) 56(84) bytes of data.
64 bytes from 172.19.0.4: icmp_seq=1 ttl=64 time=0.062 ms
```

Sabendo então qual a subnet utiliza pelo Kind, o meu arquivo de ```ConfigMap``` ficará conforme mostrado abaixo, onde reservo 5 IPs.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.17.255.250-172.17.255.254
```

Basta aplicar a configuração.
```bash
kubectl create -f ConfigMap.yml
```

MetalLB devidamente configurado e funcional.