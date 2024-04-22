# Предоставление ресурсов
В данной лабораторки мы займеся подготовкой кластера kubernetes
# Настройка БД серверов
Мы настроим тестовый файл который будет хранить в себе адресса всех серверов кластера Kubernetes
~~~
IPV4_ADDRESS FQDN HOSTNAME POD_SUBNET
~~~
Каждая колонка хранить в себе информацию IP address `IPV4_ADDRESS`, полное доменное имя `FQDN (fully qualified domain name)`, имя хоста `HOSTNAME`, а так же IP subnet `POD_SUBNET`. Kubernetes назначает один IP-адрес каждому «поду», а «POD_SUBNET» представляет собой уникальный диапазон IP-адресов, назначенный для этого каждой машине в кластере.

Вот пример базы данных компьютеров, аналогичный той, которую мы будем использовать. Каждая машина доступна друг от друга и из «admin».
~~~
XXX.XXX.XXX.XXX server.kubernetes.local server  
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1 10.200.0.0/24
XXX.XXX.XXX.XXX node-2.kubernetes.local node-2 10.200.1.0/24
XXX.XXX.XXX.XXX node-3.kubernetes.local node-3 10.200.2.0/24
XXX.XXX.XXX.XXX work-1.kubernetes.local work-1 10.200.0.0/24
XXX.XXX.XXX.XXX work-2.kubernetes.local work-2 10.200.1.0/24
XXX.XXX.XXX.XXX work-3.kubernetes.local work-3 10.200.2.0/24
~~~
# Настройка SSH доступов
Нужно настроить для каждого сервера ssh доступ для пользователя `root` из нашего текстого файла `machine.txt`

# Разрешить пользователю root SSH доступ
Так как по умолчанию на серверах Debian по умолчанию данный доступ запрещен нужно будет его включить на каждом сервере.
Для этого нам надо изменить конфигурационный файл который находиться в директории `/etc/ssh/sshd_config`
~~~
sed -i \
's/^#PermitRootLogin.*/PermitRootLogin yes/' \
/etc/ssh/sshd_config
~~~
Теперь нам только осталось перезапусть службу sshd для того что бы изменение применилсь 
~~~
systemctl restart sshd
~~~
# Генерация и доставка SSH ключа
Генерация SSH ключа
~~~
ssh-keygen
~~~
Копирование SSH ключа на сервера
~~~
while read IP FQDN HOST SUBNET; do
ssh-copy-id root@${IP}
done < machine.txt
~~~
