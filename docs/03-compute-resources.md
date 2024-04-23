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
Далее для проверки мы можем использовать следуюшию команду
~~~
while read IP FQDN HOST SUBNET; do
ssh -n root@${IP} uname -o -m
done < machina.txt
~~~
В ответе мы должны получить 
~~~
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
aarch64 GNU/Linux
~~~
# Имя хоста
В данной секции мы переименуем имя хостов `server`, `node-1`, `node-2`, `node-3`, `work-1`, `work-2` и `work-3`.
Имя хоста будет использоваться при выполнении команд из Jumpbox на каждую машину.
Имя хоста также играет важную роль в кластере.
Вместо того, чтобы клиенты Kubernetes использовали IP-адрес для отправки команд серверу API Kubernetes, эти клиенты будут использовать вместо этого имя хоста сервера. Имена хостов также используются каждой рабочей машиной.
Данные команды мы будем использоваться с сервера админимстрирование
Ранее мы настраивали текстовый файл `machines.txt` 
~~~
while read IP FQDN HOST SUBNET; do
  CMD="sed -i 's/^127.0.1.1.*/127.0.1.1/t${FQDN} ${HOST}/' /etc/hosts"
  ssh -n root@${IP} "$CMD"
  ssh -n root@${IP} hostnamectl hostname ${HOST}
done < machines.txt
~~~
Проверяем то что все правильно применилось 
~~~
while read IP FQDN HOST SUBNET; do
ssh -n root@${IP} hostname --fqdn
done < machines.txt
~~~
Должны получить ответь 
~~~
server.kubernetes.local
node-1.kubernetes.local
node-2.kubernetes.local
node-3.kubernetes.local
work-1.kubernetes.local
work-2.kubernetes.local
work-3.kubernetes.local
~~~
# DNS
В данной секции мы настроим имя хостов на сервере администрирование а потом скопируем и настроем на остальных серверах
Создадим новый файл под названием `hosts`, И добавим туда заголовок.
~~~
echo "" > hosts
echo "Kubet The Hard Way" >> hosts
~~~
Теперь скопируем наши данные с нашего текстого файла `machines.txt`, в наш новосозданный файл `hosts`
~~~
while read IP FQDN HOST SUBNET; do
ENTRY="${IP} ${FQDN} ${HOST}
echo $ENTRY >> hosts
done < machines.txt
~~~
Проверям что все скопировалось 
~~~
cat hosts
~~~
Должен быть ответ 
~~~
# Kubernetes The Hard Way
172.31.31.83 server.kubernetes.local server
172.31.22.129 node-1.kubernetes.local node-1
172.31.24.219 node-2.kubernetes.local node-2
172.31.28.247 node-3.kubernetes.local node-3
172.31.17.24 work-1.kubernetes.local work-1
172.31.29.109 work-2.kubernetes.local work-2
172.31.17.34 work-3.kubernetes.local work-3
172.31.31.83 server.kubernetes.local server
172.31.22.129 node-1.kubernetes.local node-1
172.31.24.219 node-2.kubernetes.local node-2
172.31.28.247 node-3.kubernetes.local node-3
172.31.17.24 work-1.kubernetes.local work-1
172.31.29.109 work-2.kubernetes.local work-2
172.31.17.34 work-3.kubernetes.local work-3
~~~
# Применяем на локальной машине 
Теперь нам надо скопировать с файла `hosts` в `/etc/hosts` 
~~~
cat hosts >> /etc/hosts
~~~
Проверяем все ли правильно
~~~
127.0.0.1       localhost
127.0.1.1       jumpbox

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

# Kubernetes The Hard Way
XXX.XXX.XXX.XXX server.kubernetes.local server
XXX.XXX.XXX.XXX node-0.kubernetes.local node-0
XXX.XXX.XXX.XXX node-1.kubernetes.local node-1
~~~
К этому моменту сервера должны быть настроеные так что бы могли общаться по SSH
# Добавляем DNS  записи на удаленные сервера
~~~
while read IP FQDN HOST SUBNET; do
scp hosts root@${IP}:~/
ssh -n root@${HOST} "cat hosts >> /etc/hosts"
done < machines.txt
~~~

Далее: [Создание сертификатов](04-certificate-authority.md)
