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

