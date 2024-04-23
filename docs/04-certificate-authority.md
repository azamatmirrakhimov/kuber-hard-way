# Настроем etcd кластер
Для того что бы настроить etcd кластер нам нужно сгенерировать сертификаты
1 `ca-config.json`  `ca-csr.json`  `ca-key.pem`  `ca.csr`  `ca.pem`
И сертификаты kuber
2 `kubernetes-csr.json`  `kubernetes-key.pem`  `kubernetes.csr`  `kubernetes.pem`
### 1 Генерация CA
~~~
{

cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
~~~

### 2 `kubernetes-csr.json`  `kubernetes-key.pem`  `kubernetes.csr`  `kubernetes.pem`
Нужно создать переменную
~~~
CERT_HOSTNAME=10.32.0.1,<controller node 1 Private IP>,<controller node 1 hostname>,<controller node 2 Private IP>,<controller node 2 hostname>,<API load balancer Private IP>,<API load balancer hostname>,127.0.0.1,localhost,kubernetes.default
~~~
Теперь можно приступать к генерации сертификатов
~~~
{

cat > kubernetes-csr.json << EOF
{

  "CN": "kubernetes",
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
  -hostname=${CERT_HOSTNAME} \
  -profile=kubernetes \
  kubernetes-csr.json | cfssljson -bare kubernetes

}
~~~
Копируем сертификаты на сервера
~~~
scp ca.pem kubernetes-key.pem  kubernetes.pem root@node-1:~/
~~~
~~~
 scp ca.pem kubernetes-key.pem  kubernetes.pem root@node-2:~/
~~~
~~~
scp ca.pem kubernetes-key.pem  kubernetes.pem root@node-3:~/
~~~
Копируем etcd
~~~
scp etcd-v3.4.27-linux-amd64.tar.gz root@node-1:~/
~~~
~~~
scp etcd-v3.4.27-linux-amd64.tar.gz root@node-2:~/
~~~
~~~
scp etcd-v3.4.27-linux-amd64.tar.gz root@node-3:~/
~~~
Теперь на каждом сервере `node-1` `node-2` и `node-3` нужно будет сделать одни и те же команды
~~~
tar -xvf etcd-v3.4.27-linux-amd64.tar.gz
~~~
~~~
mv etcd-v3.4.27-linux-amd64/etcd* /usr/local/bin/
~~~
~~~
mkdir -p /etc/etcd /var/lib/etcd
~~~
~~~
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
~~~
Далее надо создать пару переменных что бы создать всеми нами любимые конфиг файлы
~~~
ETCD_NAME=<cloud server hostname>
~~~
~~~
INTERNAL_IP=<servet ip>
~~~
~~~
INITIAL_CLUSTER=<controller 1 hostname>=https://<controller 1 private ip>:2380,<controller 2 hostname>=https://<controller 2 private ip>:2380
~~~
Проверяем что все правильно
~~~
echo $ETCD_NAME
~~~
~~~
echo $INTERNAL_IP
~~~
~~~
echo $INITIAL_CLUSTER
~~~
А теперь создаем конфиг файл
~~~
cat << EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster ${INITIAL_CLUSTER} \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
~~~
Перезапускаем демон
~~~
systemctl daemon-reload
~~~
~~~
systemctl enable etcd
~~~
~~~
systemctl start etcd
~~~
~~~
systemctl status etcd
~~~
Проверяем то что все работат 
~~~
ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
~~~
В ответе должны получить
~~~
ad30d1b4177b2333, started, node-3, https://172.31.17.121:2380, https://172.31.17.121:2379, false
b1fd7172d09ca898, started, node-2, https://172.31.24.235:2380, https://172.31.24.235:2379, false
b225ba2fedf4c4a1, started, node-1, https://172.31.18.85:2380, https://172.31.18.85:2379, false
~~~


