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

# Копируем данные с репозитория
~~~
git clone --depth 1 \
  https://github.com/azamatmirrakhimov/kuber-hard-way.git
~~~
переходим в директорию
~~~
cd kubet-hard-way
~~~
Скачиваем зависимости
~~~
mkdir downloads
~~~
Проверяем ссылку скачивание командой cat 
~~~
cat download.txt
~~~
Теперь скачиваем все зависимости с помощю нашего текстового файла downloads.txt с помощю команды wget
~~~
wget -q --show-progress \
--https-only \
--timestamping \
-P downlods \
-i downlod.txt
