✅ Задание 1: Elasticsearch
Установка:

sudo apt update
sudo apt install openjdk-11-jdk -y
wget https://mirror.yandex.ru/mirrors/elastic/8/pool/main/e/elasticsearch/elasticsearch-8.11.3-amd64.deb
sudo dpkg -i elasticsearch-8.11.3-amd64.deb

Настройка:

Открываем файл:

sudo nano /etc/elasticsearch/elasticsearch.yml

Добавляем строку (замени random-cluster-name на любое уникальное имя):

cluster.name: random-cluster-name

Запускаем и проверяем:

sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch

Проверка:

curl -X GET 'localhost:9200/_cluster/health?pretty'

    📸 Сделай скрин, где будет видно cluster_name : "random-cluster-name"

✅ Задание 2: Kibana
Установка:

wget https://mirror.yandex.ru/mirrors/elastic/8/pool/main/k/kibana/kibana-8.11.3-amd64.deb
sudo dpkg -i kibana-8.11.3-amd64.deb

Настройка:

sudo nano /etc/kibana/kibana.yml

Проверь, что стоит:

server.host: "0.0.0.0"

Запуск:

sudo systemctl enable kibana
sudo systemctl start kibana

Зайди в браузере:

http://<IP-сервера>:5601/app/dev_tools#/console

И выполни:

GET /_cluster/health?pretty

    📸 Скриншот с интерфейсом Kibana и результатом запроса

✅ Задание 3: Logstash + Nginx
Установка:

sudo apt install nginx -y
wget https://mirror.yandex.ru/mirrors/elastic/8/pool/main/l/logstash/logstash-8.11.3-amd64.deb
sudo dpkg -i logstash-8.11.3-amd64.deb

Настройка Logstash:

Создаём файл:

sudo nano /etc/logstash/conf.d/nginx.conf

Содержимое:

input {
  file {
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "nginx-access-log"
  }
}

Запуск:

sudo systemctl enable logstash
sudo systemctl start logstash

    📸 Зайди в Kibana → Discover, и найди индекс nginx-access-log — сделай скрин, где видны логи

✅ Задание 4: Filebeat
Установка:

wget https://mirror.yandex.ru/mirrors/elastic/8/pool/main/f/filebeat/filebeat-8.11.3-amd64.deb
sudo dpkg -i filebeat-8.11.3-amd64.deb

Настройка:

sudo filebeat modules enable nginx
sudo filebeat setup

Проверь filebeat.yml:

sudo nano /etc/filebeat/filebeat.yml

Убедись в наличии:

output.elasticsearch:
  hosts: ["http://localhost:9200"]

Отключи Logstash (если использовался):

sudo systemctl stop logstash

Запуск Filebeat:

sudo systemctl enable filebeat
sudo systemctl start filebeat

    📸 Открой Kibana → Discover, убедись что логи поступают через Filebeat

⭐ Задание 5* (по желанию): Логи любого другого сервиса
Пример:

Допустим, берём лог-файл myapp.log со своим форматом.

    Настрой Logstash для разбора этого формата (например, JSON):

input {
  file {
    path => "/var/log/myapp.log"
    start_position => "beginning"
    codec => "json"
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "myapp-log"
  }
}

    Перезапуск Logstash:

sudo systemctl restart logstash

    Открой Kibana → Discover → myapp-log

    📸 Скриншот с логами другого приложения, укажи его название