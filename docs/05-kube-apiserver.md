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
