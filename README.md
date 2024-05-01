# Домашнее задание к занятию "`ELK`" - `Тен Денис`


## Дополнительные ресурсы

При выполнении задания используйте дополнительные ресурсы:
- [docker-compose elasticsearch + kibana](11-03/docker-compose.yaml);
- [поднимаем elk в docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html);
- [поднимаем elk в docker с filebeat и docker-логами](https://www.sarulabs.com/post/5/2019-08-12/sending-docker-logs-to-elasticsearch-and-kibana-with-filebeat.html);
- [конфигурируем logstash](https://www.elastic.co/guide/en/logstash/7.17/configuration.html);
- [плагины filter для logstash](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html);
- [конфигурируем filebeat](https://www.elastic.co/guide/en/beats/libbeat/5.3/config-file-format.html);
- [привязываем индексы из elastic в kibana](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html);
- [как просматривать логи в kibana](https://www.elastic.co/guide/en/kibana/current/discover.html);
- [решение ошибки increase vm.max_map_count elasticsearch](https://stackoverflow.com/questions/42889241/how-to-increase-vm-max-map-count).

**Примечание**: если у вас недоступны официальные образы, можете найти альтернативные варианты в DockerHub, например, [такой](https://hub.docker.com/layers/bitnami/elasticsearch/7.17.13/images/sha256-8084adf6fa1cf24368337d7f62292081db721f4f05dcb01561a7c7e66806cc41?context=explore).

### Задание 1. Elasticsearch 

Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный. 

*Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name*.

### Решение Задание 1. Elasticsearch 

Устанавливаем Elasticsearch

```bash
apt update && apt install gnupg apt-transport-https <--зависимости 
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add - <--добавляем gpg-ключ 
echo "deb [trusted=yes] https://mirror.yandex.ru/mirrors/elastic/7/ stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list <--добавляем репозиторий в apt 
apt update && apt-get install elasticsearch <--устанавливаем elastic 
systemctl daemon-reload <--обновляем конфиги systemd 
systemctl enable elasticsearch.service <--включаем юнит 
systemctl start elasticsearch.service <--запускаем сервис
```
Проверяем установку и запуск Elasticsearch
```
curl 'localhost:9200/_cluster/health?pretty'
```
![image](https://github.com/killakazzak/11-03-sdb-hw/assets/32342205/a657f7b3-208f-427d-b397-b2966c8e2533)

Настраиваем Elasticsearch

```
vim  /etc/elasticsearch/elasticsearch.yml
```
```yaml
cluster.name: netology-logging <--меняем имя кластера
node.name: node-1 <--меняем название ноды
node.roles: [ master, data, ingest ] <--какую функцию будет выполнять эта нода
cluster.initial_master_nodes: ["node-1"] <--узлы, участвующие в голосовании по выбору мастера
discovery.seed_hosts: ["10.159.86.95"] <--список возможных мастеров кластера
path.data: /var/lib/elasticsearch <-где храним данные
```
Проверяем изменения конфигурации

```bash
systemctl restart elasticsearch
curl 'localhost:9200/_cluster/health?pretty'
```
![image](https://github.com/killakazzak/11-03-sdb-hw/assets/32342205/c4aa9693-bf1a-40b6-953f-ffb29c7f523d)




---

### Задание 2. Kibana

Установите и запустите Kibana.

*Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty*.

### Решение Задание 2. Kibana

Устанавливаем Kibana

```bash
apt install kibana <--установка
systemctl daemon-reload <--обновляем конфиги systemd
systemctl enable kibana.service <--включаем юнит
systemctl start kibana.service <--запускаем сервис
```
Настраиваем Kibana

```
vim /etc/kibana/kibana.yml
```
```yaml
server.host: "0.0.0.0"
```
```
systemctl restart kibana
systemctl status kibana
```

Проверяем установку Kibana

![image](https://github.com/killakazzak/11-03-sdb-hw/assets/32342205/1bc440c9-54dd-4a0c-9938-461da7dbbb40)

![image](https://github.com/killakazzak/11-03-sdb-hw/assets/32342205/ce70547b-9769-4199-9c00-69f5bcca1e1d)



---

### Задание 3. Logstash

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch. 

*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.*

### Решение Задание 3. Logstash

Устанавливаем Logstash и nginx

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add - <--добавляем gpg-ключ 
echo "deb [trusted=yes] https://mirror.yandex.ru/mirrors/elastic/7/ stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list <--добавляем репозиторий в apt 
apt update && apt install logstash <--устанавливаем logstash
apt install nginx
systemctl daemon-reload <--обновляем конфиги systemd
systemctl enable logstash.service <--включаем юнит
systemctl start logstash.service <--запускаем сервис
systemctl enable nginx <--включаем юнит
systemctl start nginx <--запускаем сервис
```
Настраиваем Logstash

```
vim /etc/logstash/conf.d/logstash.conf
```

```yaml
input {
  file {
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
  }
}


output {
  elasticsearch {
     hosts => ["localhost:9200"]
     index    => "nginx-logs-%{+YYYY.MM.dd}"
  }
}
```


Проверка

![image](https://github.com/killakazzak/11-03-sdb-hw/assets/32342205/fed7d9bf-ecdd-44f1-aca9-b8fa888d7ce6)

---

### Задание 4. Filebeat. 

Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat. 

*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.*

### Решение Задание 4. Filebeat. 

Устанавливаем Filebeat

```bash
apt install filebeat <--установка
systemctl daemon-reload <--обновляем конфиги systemd
systemctl enable filebeat.service <--включаем юнит
systemctl start filebeat.service <--запускаем сервис
```
Настраиваем Logstash для приема логов с Filebeat
```bash
vim /etc/logstash/conf.d/logstash.conf
```
logstash.conf
```yaml
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
     hosts => ["10.159.86.95:9200"]
     index    => "nginx-logs-%{+YYYY.MM.dd}"
  }
}
```
Настраиваем Filebeat для отправки в Logstash:
```bash
vim /etc/filebeat/filebeat.yml
```
filebeat.yml

```yaml
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log

output.logstash:
  hosts: ["10.159.86.95:5044"]

processors:
  - drop_fields:
      fields: ["beat", "input_type", "prospector", "input", "ecs"]
```

Проверка
![image](https://github.com/killakazzak/11-03-sdb-hw/assets/32342205/e0fb4389-0047-411d-af8f-d42cb16c3802)


## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 5*. Доставка данных 

Настройте поставку лога в Elasticsearch через Logstash и Filebeat любого другого сервиса , но не Nginx. 
Для этого лог должен писаться на файловую систему, Logstash должен корректно его распарсить и разложить на поля. 

*Приведите скриншот интерфейса Kibana, на котором будет виден этот лог и напишите лог какого приложения отправляется.*

### Решение Задание 5*. Доставка данных 






Настройка отправку и получение логов Apache через Filebeate и Logstash

logstash.conf

```yaml
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
     hosts => ["10.159.86.95:9200"]
     index    => "apache-logs-%{+YYYY.MM.dd}"
  }
}
```

filebeat.yml

```yaml
- type: log
  enabled: true
  paths:
    - /var/log/apache2/access.log

output.logstash:
  hosts: ["10.159.86.95:5044"]

processors:
  - drop_fields:
      fields: ["beat", "input_type", "prospector", "input", "ecs"]
```

Проверка получения логов Apache через filebeat
![image](https://github.com/killakazzak/11-03-sdb-hw/assets/32342205/b1423fd5-1ed1-4453-a5a7-b2ff27e36459)

