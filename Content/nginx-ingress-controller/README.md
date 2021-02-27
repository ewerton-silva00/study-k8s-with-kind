## Instalação e configuração do Nginx Ingress Controller

Aqui faremos a implantação do famigerado [**Ingress Controller**](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) chamado [**Nginx Ingress Controller**](https://kubernetes.github.io/ingress-nginx/).

Dessa forma podemos trabalhar com recursos do tipo [**Ingress**](https://kubernetes.io/docs/concepts/services-networking/ingress/).

A implantação default do **Nginx Ingress Controller** é bem simples. Como estamos utilizando o Kind, usaremos o tipo de instalação [**Bare Metal**](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal).

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.35.0/deploy/static/provider/baremetal/deploy.yaml
```