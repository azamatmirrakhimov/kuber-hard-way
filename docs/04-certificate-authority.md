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
~~~
CERT_HOSTNAME=10.32.0.1,<controller node 1 Private IP>,<controller node 1 hostname>,<controller node 2 Private IP>,<controller node 2 hostname>,<API load balancer Private IP>,<API load balancer hostname>,127.0.0.1,localhost,kubernetes.default
~~~
