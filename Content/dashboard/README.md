## Instalação e configuração do Kubernetes Dashboard

O [**Kubernetes Dashboard**](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) é uma interface de usuário baseada na WEB na qual é possível gerenciar o cluster Kubernetes e os aplicativos implantados.

A implantação desse componente é bem simples.

```bash
kubectl create --filename https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Crie o usuário ```admin-user``` com suas respectivas permissões:

```yaml
cat <<'EOF' | kubectl create --filename -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

Para acessar o dashboard é preciso recuperar o token do usuário ```admin-user```. Execute o comando abaixo e copie o token.

```bash
kubectl --namespace kubernetes-dashboard describe secret $(kubectl --namespace kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

Inicialize o proxy para acesso ao dashboard com o comando ```kubectl proxy```.

```
➜ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

No browser informe o endereço abaixo e insira o token do usuário.
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Kubernetes Dashboard implantado e funcional.