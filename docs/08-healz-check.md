# Настройка nginx для healthz check
Установка nginx  сервера
~~~
sudo apt-get install -y nginx-full
~~~
### Создание конфигурации nginx для проверки health check proxy:
~~~
cat > kubernetes.default.svc.cluster.local << EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
    proxy_pass                    https://127.0.0.1:6443/healthz;
    proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
~~~
### Переместим наши настройки в директорию nginx что бы он использовал наши настройки:
~~~
sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
~~~
~~~
sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
~~~
Перезапускаем nginx
~~~
sudo systemctl restart nginx
~~~
Добавляем nginx  в автозапуск
~~~
sudo systemctl enable nginx
~~~
### На этом мы все настроили проверяем:
~~~
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
~~~
### В ответе должны получить 200 OK.

# Настройка RBAC for Kubelet Authorization 

### Содадим роль с необходиммами провами:
~~~
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
~~~
### Передаем прова:
~~~
cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
~~~

Далее: [LoadBalancer](09-loadbalancer.md)
