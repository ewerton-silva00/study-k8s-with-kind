## Instalação e configuração do Nginx Ingress Controller

Aqui faremos a implantação do [**Ingress Controller**](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) chamado [**Nginx Ingress Controller**](https://kubernetes.github.io/ingress-nginx/).

Dessa forma podemos trabalhar com recursos do tipo [**Ingress**](https://kubernetes.io/docs/concepts/services-networking/ingress/).

A implantação default do **Nginx Ingress Controller** é bem simples. Como estamos utilizando o Kind, usaremos o tipo de instalação [**Bare Metal**](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal).

Instale o repositório contendo o Helm Chart.
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

Instale o `Nginx Ingress Controller`.
```bash
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  --version 4.0.9 \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.nodeSelector.loadbalancer=enabled \
  --set "controller.tolerations[0].key=node-role.kubernetes.io/master" \
  --set "controller.tolerations[0].effect=NoSchedule" \
  --set podLabels.loadbalancer=enabled \
  --set "service.annotations.metallb.universe.tf/address-pool=default" \
  --set defaultBackend.enabled=true \
  --set defaultBackend.image.repository=rafaelperoco/default-backend,defaultBackend.image.tag=1.0.0
kubectl wait --for condition=Available=True deploy/ingress-nginx-controller --namespace ingress-nginx --timeout -1s
```

Verifique se o serviço `ingress-nginx-controller` adotou um IP do pool configurado no `MetalLB`.
```bash
kubectl get service --namespace ingress-nginx ingress-nginx-controller
```

O resultado será semelhante a este abaixo:
```
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.96.71.17   172.18.0.10   80:30440/TCP,443:30709/TCP   21m
```

No browser, informe o IP mostrado no `EXTERNAL-IP` e tu será redirecionado para um `defaultBackend` que apenas mostra uma tela com 404. Está funcional.
