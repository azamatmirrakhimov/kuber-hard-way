# Настройка сервера алминистрирование

В данном разделе мы настроим машину для админимстрирование кластером кубера
Все команды будем делать от имени администратора
### Install Utilitis
~~~
apt-get -y install wget curl vim openssl git
~~~
# cfssl:
~~~
wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
~~~
~~~
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
~~~
~~~
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
~~~
~~~
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
~~~
~~~
cfssl version
~~~
