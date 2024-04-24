# Настройки кубе контроллер манагера

Нам нужно создать кубе конфиг файл
1. Но для этого сначала сгенирирум сертификаты `kube-controller-manager.pem`
~~~
{

cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
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
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
~~~
2. Создаем сам `kube-controller-manager.kubeconfig`
~~~
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
~~~
После того как мы его создадим нам нужно переместить на все сервера node-1, node-2 и node-3
~~~
scp kube-controller-manager.kubeconfig node-1:~/
~~~
Теперь нам надо залогиниться на сервер node-1, остальные все действие нужно пофторить да остальных серверах
Копирум наш кубе конфиг в директорию библиотеке кубернетес
~~~
cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
~~~
Создаем сервисный файл для управление
~~~
cat << EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
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
Добавляем наш сервис в автозапуск
~~~
systemctl enable kube-controller-manager
~~~
Стартуем сервис
~~~
systemctl start kube-controller-manager
~~~
Проверяем статус
~~~
systemctl status kube-controller-manager
~~~
Если будут ошибки удобно проверяем и перенастроиваем
~~~
journalctl -u kube-controller-manager.service --no-pager --reverse | head -n 20
~~~

Далее: [Создание кластера Kube Scheduler](07-kube-scheduler.md)
