# docker #

Весь процесс включает в себя 3 этапа:
  - установка Docker Compose
  - установка Prometheus
  - увтановка VictoriaMetrics

**Основные условия, необходимые при установке Linux Oracle.**

✓ Оперативной памяти больше 4096МБ

✓ Английский язык

✓ Больше двух ядер


Теперь переходим к введению команд.
Клонируем репозиторий. 

В каталоге с репозиторием создаем каталог для конфигурации Grafana внутри общей директории

Далее создаем  директорию для Prometheus, Grafana, Promtail и Loki. Меняем их владельца на текущего пользователя, создаем файл `grafana.ini`
и из директории `config` копируем файлы в новый каталог конфигурации. 

Переименовываем   `grafana.yaml` в `docker-compose.yaml`.

После этого устанавливаем утилиту `curl` и используем ее. Устанавливаем Docker Compose последней версии и сохраняем в `/usr/bin/docker-compose`
Предоставляем права на выполнение `docker-compose`.
Устанавливаем `wget`, скачиваем файл репозитория Docker CE для CentOS и размещаем его в директории `/etc/yum.repos.d/`.
Устанавливаем Docker Engine (docker-ce), клиентские инструменты Docker (docker-ce-cli) и контейнерный рантайм containerd.
Включаем, запускаем службу Docker.
В фоновом режиме запускаем сервисы из`docker-compose.yaml` .





    git clone https://github.com/skl256/grafana_stack_for_docker.git && (добавляем сюда y)
    cd grafana_stack_for_docker && 
    sudo mkdir -p /mnt/common_volume/swarm/grafana/config && 
    sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data} && 
    sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana} && 
    touch /mnt/common_volume/grafana/grafana-config/grafana.ini && 
    cp config/* /mnt/common_volume/swarm/grafana/config/ && 
    mv grafana.yaml docker-compose.yaml && 




Устанавливаем последнею версию и утилитю docker-compose
sudo yum install curl && \
COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d" -f4) && \
sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose && \
sudo chmod +x /usr/bin/docker-compose && \
docker-compose --version && \
Установка и настройка Docker
sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce
sudo systemctl enable docker --now
docker compose up -d
Конфигурация отдельных сервисов
Prometheus
После Установки докера пишем sudo vi docker-compose.yaml (Обязательно что бы был расширения.yaml)Нас перекидывает на текстовый редактор:
Нас перекинет в текстовый редактор
Что-бы что-то изменить в тесковом редакторе нажмите insert на клавиатуре
Что бы сохранить что-то в этом документе нажимаем Esc пишем :wq! В этом текставом редакторе мы должны поставить node-exporter после services
  node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    container_name: exporter < запомнить название name
    hostname: exporter
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    ports:
      - 9100:9100
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
Далее пишем

cd
cd /mnt/common_volume/swarm/grafana/config 
sudo vi prometheus.yaml 
В этом файле нужно изменить первый ip Адрес на тот, который мы должны были запомнить
Вставляем в первый targets
После двоеточие цифры оставляем
Grafana
переходим на сайт localhost:3000

User & Password GRAFANA: admin

Код графаны: 3000
Код прометеуса: http://prometheus:9090
в меню выбираем вкладку Dashboards и создаем Dashboard

ждем кнопку +Add visualization, а после "Configure a new data source"
выбираем Prometheus
Connection

http://prometheus:9090
Authentication

Basic authentication
User: admin
Password: admin
Нажимаем на Save & test и должно показывать зелёную галочку
в меню выбираем вкладку Dashboards и создаем Dashboard

ждем кнопку "Import dashboard"
Find and import dashboards for common applications at grafana.com/dashboards: 1860 //ждем кнопку Load
Select Prometheus ждем кнопку "Import"
Note

Всё должно выглядить как на скриншоте https://github.com/TymkaTy/Docker/blob/main/grafana_stack_for_docker/Screenshot_Prometheus.png

Если не работает то переходим в

cd grafana_stack_for_docker
sudo docker compose stop
sudo docker compose up -d
То есть перезапускаем докер

VicroriaMetrics
Для начала изменим docker-compose.yaml:
cd grafana_stack_for_docker
sudo vi docker-compose.yaml
После чего в в самом текстовом редакторе после prometheus вставляем

  vmagent:
    container_name: vmagent
    image: victoriametrics/vmagent:v1.105.0
    depends_on:
      - "victoriametrics"
    ports:
      - 8429:8429
    volumes:
      - vmagentdata:/vmagentdata
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - "--promscrape.config=/etc/prometheus/prometheus.yml"
      - "--remoteWrite.url=http://victoriametrics:8428/api/v1/write"
    restart: always
  # VictoriaMetrics instance, a single process responsible for
  # storing metrics and serve read requests.
  victoriametrics:
    container_name: victoriametrics
    image: victoriametrics/victoria-metrics:v1.105.0
    ports:
      - 8428:8428
      - 8089:8089
      - 8089:8089/udp
      - 2003:2003
      - 2003:2003/udp
      - 4242:4242
    volumes:
      - vmdata:/storage
    command:
      - "--storageDataPath=/storage"
      - "--graphiteListenAddr=:2003"
      - "--opentsdbListenAddr=:4242"
      - "--httpListenAddr=:8428"
      - "--influxListenAddr=:8089"
      - "--vmalert.proxyURL=http://vmalert:8880"
    restart: always
Сохраняем и выходим

Захом в connection
там где мы писали http:prometheus:9090 пишем http:victoriametrics:9090 И заменяем имя из "Prometheus-2" в "Vika"
нажимаем на dashboards add visualition выбираем "Vika"
снизу меняем на "cod"
Переходим в терминал и пишем
echo -e "# TYPE OILCOINT_metric1 gauge\nOILCOINT_metric1 0" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus  
curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric1'

(Значение 0 меняем на любое другое)

Копируем переменную OILCOINT_metric1 и вставляем в cod
Нажимаем run image
Должно получится вот так
