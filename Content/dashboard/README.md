## Instalação e configuração do Kubernetes Dashboard

O [`Kubernetes Dashboard`](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) é uma interface de usuário baseada na WEB na qual é possível gerenciar o cluster Kubernetes e os aplicativos implantados.

Implante os componentes do `kubernetes-dashboard` utilizando `helm`.

```
➜ helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
Release "kubernetes-dashboard" does not exist. Installing it now.
NAME: kubernetes-dashboard
LAST DEPLOYED: Mon Nov 11 16:25:04 2024
NAMESPACE: kubernetes-dashboard
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
*************************************************************************************************
*** PLEASE BE PATIENT: Kubernetes Dashboard may need a few minutes to get up and become ready ***
*************************************************************************************************

Congratulations! You have just installed Kubernetes Dashboard in your cluster.

To access Dashboard run:
  kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443

NOTE: In case port-forward command does not work, make sure that kong service name is correct.
      Check the services in Kubernetes Dashboard namespace using:
        kubectl -n kubernetes-dashboard get svc

Dashboard will be available at:
  https://localhost:8443
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

Crie o token de acesso:

```bash
kubectl create token admin-user --namespace kubernetes-dashboard
```

Inicialize o proxy para acesso ao dashboard com o comando ```kubectl proxy```.

```
➜ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

No browser informe o endereço abaixo e insira o token do usuário.

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard-kong-proxy:443/proxy/#/login
```

Kubernetes Dashboard implantado e funcional.
