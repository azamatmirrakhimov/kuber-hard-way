# Начинаем настройку kublet на серверах `work-1` `work-2` и `work-3`
### 1 Шаг создадим нужные сертификаты для `workers`  на сервере `jumpbox`
Создвдим переменные с нашами даннами серверов `workers`
~~~
WORKER1_HOST=work-1
~~~
~~~
WORKER1_IP=172.31.24.130
~~~
~~~
WORKER2_HOST=work-2
~~~
~~~
WORKER2_IP=172.31.18.157
~~~
~~~
WORKER3_HOST=work-3
~~~
~~~
WORKER3_IP=172.31.22.46
~~~

Теперь можем создать сертификаты
~~~
{
cat > ${WORKER1_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER1_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
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
  -hostname=${WORKER1_IP},${WORKER1_HOST} \
  -profile=kubernetes \
  ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}
  
cat > ${WORKER2_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER2_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
     "C": "US",
     "L": "Portland",
     "O": "system:nodes",
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
 -hostname=${WORKER2_IP},${WORKER2_HOST} \
 -profile=kubernetes \
 ${WORKER2_HOST}-csr.json | cfssljson -bare ${WORKER2_HOST}
	
cat > ${WORKER3_HOST}-csr.json << EOF
{
  "CN": "system:node:${WORKER3_HOST}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
     "C": "US",
     "L": "Portland",
     "O": "system:nodes",
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
 -hostname=${WORKER3_IP},${WORKER3_HOST} \
 -profile=kubernetes \
 ${WORKER3_HOST}-csr.json | cfssljson -bare ${WORKER3_HOST}
 }
~~~
Так же нам нужно создать `kubeconfig`

~~~

for instance in work-1 work-2 work-3; do
 >   kubectl config set-cluster kubernetes-the-hard-way \
     --certificate-authority=ca.pem \
     --embed-certs=true \
     --server=https://${KUBERNETES_ADDRESS}:6443 \
     --kubeconfig=${instance}.kubeconfig
   kubectl config set-credentials system:node:${instance} \
     --client-certificate=${instance}.pem \
     --client-key=${instance}-key.pem \
     --embed-certs=true \
     --kubeconfig=${instance}.kubeconfig
   kubectl config set-context default \
     --cluster=kubernetes-the-hard-way \
     --user=system:node:${instance} \
     --kubeconfig=${instance}.kubeconfig

   kubectl config use-context default --kubeconfig=${instance}.kubeconfig
 done
~~~
Теперь нам надо все доставить на сервера для это воспользуемся `scp`

~~~
scp \
ca.pem work-1-key.pem work-1.pem work-2-key.pem work-2.pem work-3-key.pem work-3.pem \
work-1.kubeconfig work-2.kubeconfig work-3.kubeconfig \
work-1:~/
~~~
Те же действия на остальных `work-2` и `work-3` только не забываем копировать ключи правильном именем 
# Теперь мы полкючаемся к наши `workers` серверам и начинаем настройку
~~~
ssh work-1
~~~
Все действия надо будет повторить на двух остальных серверах
### Добавляем переменную `host`
~~~
HOSTNAME=$(hostname)
~~~
Копируем ключи в директорию `/var/lib/kubelet/`
~~~
mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
~~~
Копируем `kubeconfig`  директорию `/var/lib/kubelet/kubeconfig`
~~~
mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
~~~
Создаем `yaml` конфиг файл
~~~
cat << EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
~~~
Теперь мы можем создать сервисный файл
~~~
cat << EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --register-node=true \\
  --v=2 
  --hostname-override=${HOSTNAME} \\
  --allow-privileged=true
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
~~~
Перезапускаем `daemon`
~~~
systemctl daemon-reload 
~~~
Добавляем сервис в автозапуск
~~~
systemctl enable kubelet.service 
~~~
Стартуем сервис
~~~
systemctl start kubelet.service 
~~~
Проверяем
~~~
systemctl status kubelet.service 
~~~
Если ловим ошибки проверем через `journalctl`
~~~
journalctl -u kubelet.service --no-pager --reverse | head -n 20
~~~

Далее
