# На данном уроке мы настроим удаленный доступ для управление
### Так как проблемма в том что у `cloud guru` адресса всегда меняются и нету доступа до `6443` порта
### Мы наебем систему
И вот как мы это сделаем откроем 2 терминала в одном пропишем команду проксирование
~~~
ssh -L 6443:localhost:6443 root@server
~~~
И пока второй терминал залогиненый по порту 6443 на сервер балансировщика мы можем работать

# Для начала мы создадим для себя сертификаты администрирование
Данный сертификат мы настраиваем на сервере `jumpbox`

~~~
{

cat > admin-csr.json << EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
admin-csr.json | cfssljson -bare admin

}
~~~
Теперь нам надо создать новый кластер для этого нам нужен наш сертификат `ca.pem` название кластера будет `kubernetes-the-hard-way`
~~~
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://localhost:6443
~~~
Ответ
~~~
Cluster "kubernetes-the-hard-way" set.
~~~
Теперь мы устанавливаем наши учетные данные то есть пользователя `admin` и его сертификаты `admin.pem` `admin-key.pem`
~~~
kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem

kubectl config set-context kubernetes-the-hard-way \
  --cluster=kubernetes-the-hard-way \
  --user=admin
~~~
Ответ
~~~
User "admin" set.
~~~

~~~
kubectl config use-context kubernetes-the-hard-way
~~~
Ответ
~~~
Switched to context "kubernetes-the-hard-way".
~~~
~~~
kubectl get pods
~~~
Ответ
~~~
error: the server doesn't have a resource type "version"
~~~
~~~
kubectl get nodes
~~~
Ответ
~~~
NAME     STATUS   ROLES    AGE     VERSION
work-1   Ready    <none>   4h57m   v1.28.3
work-2   Ready    <none>   5h12m   v1.28.3
work-3   Ready    <none>   8h      v1.28.3
~~~
~~~
kubectl version
~~~
Ответ
~~~
Client Version: v1.28.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.3
~~~

Далее: [Настройка сети](RemoteAccess/02-networking.md)
