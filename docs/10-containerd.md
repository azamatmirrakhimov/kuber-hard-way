# Начинаем настройку наших `worker nodes`  запускаем `containerd` сервис

### Копируем с нашего сервера `jumpbox` на все `worker` файлы из папки `downloads`
~~~
for host in work-1 work-2 work-3; do
  scp \
    downloads/runc.amd64 \
    downloads/crictl-v1.28.0-linux-amd64.tar.gz \
    downloads/cni-plugins-linux-amd64-v1.3.0.tgz \
    downloads/containerd-1.7.8-linux-amd64.tar.gz \
    downloads/kubectl \
    downloads/kubelet \
    downloads/kube-proxy \
    root@$host:~/
done
~~~
Далее переходим на наши сервера и устанавливаем зависимости
~~~
{
  apt-get update
  apt-get -y install socat conntrack ipset
}
~~~
Проверяем swap 
~~~
swapon --show
~~~
Если после данной команды не чего не получем то swap отключен, Если получим какой либо ответ используем команду для отключение swap
~~~
swapoff -a
~~~
Создаем инсталиционные директории
~~~
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
~~~
Устанавлиеваем worker зависемости
~~~
{
  mkdir -p containerd
  tar -xvf crictl-v1.28.0-linux-amd64.tar.gz
  tar -xvf containerd-1.7.8-linux-amd64.tar.gz -C containerd
  tar -xvf cni-plugins-linux-amd64-v1.3.0.tgz -C /opt/cni/bin/
  mv runc.amd64 runc
  chmod +x crictl kubectl kube-proxy kubelet runc 
  mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
  mv containerd/bin/* /bin/
}
~~~

Настроиваем `bridge` сетевое окружение:
Создаем файл
~~~
vi 10-bridge.conf
~~~
В нем делаем записи
~~~
{
  "cniVersion": "1.0.0",
  "name": "bridge",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "ranges": [
      [{"subnet": "SUBNET"}]
    ],
    "routes": [{"dst": "0.0.0.0/0"}]
  }
}
~~~
Создаем файл
~~~
vi 99-loopback.conf 
~~~
Добавляем запись
~~~
{
  "cniVersion": "1.1.0",
  "name": "lo",
  "type": "loopback"
}
~~~
Перемешаем настройки в папку конфигурации
~~~
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
~~~
