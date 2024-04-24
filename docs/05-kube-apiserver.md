# Для создание Kube API нам понадобиться `1-сертификаты` `2-bin` файлы настройка `3-сервиса`

### Содание сертификатов
Выполнем команду
~~~
{

cat > service-account-csr.json << EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
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
  service-account-csr.json | cfssljson -bare service-account

}
~~~
У нас должны создаться файлы
~~~
service-account-csr.json service-account-key.pem service-account.csr service-account.pem
~~~
Теперь нам осталось создать `service-account.key`
~~~
openssl genrsa -out service-account.key 2048
~~~
Осталось создать метод шифрование `encryption-config.yaml`
~~~
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

cat > encryption-config.yaml << EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
~~~
Теперь нам надо доставить `bin` и `сертификаты` на наши сервера
начинаем с бинарных файлов
~~~
scp kube-apiserver kube-controller-manager kube-scheduler kubectl node-1:~/
~~~
И так же на два остальных сервера
Копируем сертификаты
~~~
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem service-account.key \
  encryption-config.yaml node-1:~/
~~~
Те же действия для двух остальных серверов
### Настройка на серверах 
Я сделаю настройки на одном сервере `node-1` на останых нужно сделать те же действия `node-2` и `node-3`
Создаем папку для хранение настроек кубера далее она нам понадобиться
~~~
mkdir -p /etc/kubernetes/config
~~~
Даем разрешение на запись для бинарных файлов
~~~
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
~~~
И перемещаем их в папку где храняться все бинарные файлы
~~~
mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
~~~
Создаем папку для хранение наших сертификатов
~~~
mkdir -p /var/lib/kubernetes/
~~~
И копируем все наши сертификаты в даную папку
~~~
cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem service-account.key \
  encryption-config.yaml /var/lib/kubernetes/
~~~
Теперь нам надо только создать сервис файл что бы мы могли его запускать с помощю стандартной команды linux
Но перед этим нам надо сделать несколько переменных
~~~
INTERNAL_IP=private-ip
~~~
~~~
CONTROLLER1_IP=172.31.18.85
~~~
~~~
CONTROLLER2_IP=172.31.24.235
~~~
~~~
CONTROLLER3_IP=172.31.17.121
~~~
Создание сервисного файла
~~~
cat << EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --enable-swagger-ui=true \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://$CONTROLLER0_IP:2379,https://$CONTROLLER1_IP:2379 \\
  --event-ttl=1h \\
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2 \\
  --kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
~~~


