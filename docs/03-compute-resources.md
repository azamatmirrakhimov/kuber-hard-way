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

