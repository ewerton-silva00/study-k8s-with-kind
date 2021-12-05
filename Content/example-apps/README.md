# Aplicação de Exemplo

Aqui disponibilizo de forma rápida uma aplicação de fácil instalação e que proporciona um cenário bom para estudos sobre Kubernetes.

**Rocket.Chat**

O [`Rocket.Chat`](https://rocket.chat/) é uma plataforma de comunicação de código aberto. Uma alternativa ao Slack.

Instale o repositório contendo o Helm Chart.
```bash
helm repo add rocketchat https://rocketchat.github.io/helm-charts
```

Instale o `Rocket.Chat`.
```bash
helm upgrade --install rocketchat rocketchat/rocketchat \
    --version 3.1.0 \
    --create-namespace \
    --namespace rocketchat \
    --values rocketchat/rocketchat.yaml
kubectl wait --for condition=Available=True deploy/rocketchat-rocketchat --namespace rocketchat --timeout -1s
kubectl wait --for condition=ready pod/rocketchat-mongodb-0 --namespace rocketchat --timeout -1s
```
> Confira o arquivo `rocketchat/rocketchat.yaml` para entender ou modificar as configurações.

No arquivo `rocketchat/rocketchat.yaml` o parâmetro `host` recebe o valor `chat.meudominio.local`. Sendo assim, você precisa adicionar no `/etc/host` do seu computador a entrada abaixo:

```
172.18.0.10 chat.meudominio.local
```
> O IP acima, `172.18.0.10` é o IP atribuído ao LoadBalancer do Nginx Ingress Controller.

Para consultar qual IP atribuído ao LoadBalancer do Nginx Ingress Controller execute o comando abaixo:

```bash
kubectl get service --namespace ingress-nginx ingress-nginx-controller
```

O resultado será semelhante a este mostrado abaixo:

```
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.96.122.185   172.18.0.10   80:31598/TCP,443:30455/TCP   170m
```

Pronto, o IP mostrado no `EXTERNAL-IP` será o IP que você utilizará como entrada de acesso para as aplicações.
