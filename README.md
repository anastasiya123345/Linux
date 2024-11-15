# docker #

Весь процесс включает в себя 3 этапа:
  - установка Docker Compose
  - установка Prometheus
  - увтановка VictoriaMetrics

**Основные условия, необходимые при установке Linux Oracle.**

✓ Оперативной памяти больше 4096МБ

✓ Английский язык

✓ Больше двух ядер процессора

✓ Более 20ГБ места на жестком диске

Теперь переходим к введению команд в терминал.

# => Grafana Stack

1. Установка Docker'а

Установка wget для скачивания файлов.

        sudo yum install wget

![изображение](https://github.com/user-attachments/assets/0d50d629-7f1a-4f83-a895-f90f464acfaa)

Скачивание файла репозитория Docker CE для CentOS и размещение его в директории /etc/yum.repos.d/.

    sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo

![изображение](https://github.com/user-attachments/assets/18579752-b985-49aa-8a99-f05de582de9b)

Установка Docker Engine (docker-ce), клиентских инструментов Docker (docker-ce-cli) и контейнерного рантайма containerd.

        sudo yum install docker-ce docker-ce-cli containerd.io

![изображение](https://github.com/user-attachments/assets/b32cc768-d75a-4780-bee3-4ea15a57b08d)

Включение и немедленный запуск службы Docker.

        sudo systemctl enable docker --now
        
![изображение](https://github.com/user-attachments/assets/718be7f0-b330-44b4-acc8-286003bfee54)

2. Установка Compose

Установка утилиты curl (на CentOS/RHEL) для выполнения HTTP-запросов.

       sudo yum install curl
   
![изображение](https://github.com/user-attachments/assets/a1db0524-5893-48e3-b36f-d487d6f5f22a)

Использование команды curl для получения последней версии Docker Compose с GitHub API. Фильтрация ответа для извлечения тега с версией.

      COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\" -f4)

Загрузка последней версии Docker Compose с официального репозитория GitHub и сохранение её в /usr/bin/docker-compose.

      sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose

![изображение](https://github.com/user-attachments/assets/c6b0cfc3-7bb0-47ce-9287-563c2a6a008a)

Предоставление прав на выполнение файла docker-compose.

      sudo chmod +x /usr/bin/docker-compose

Проверка установленной версии Docker Compose.

      sudo docker-compose --version

![изображение](https://github.com/user-attachments/assets/e3c6ee5a-e8eb-4307-a48a-cb1b2f8fadd1)

4. Делаем Grafana

Установка git'а

        sudo yum install git

![изображение](https://github.com/user-attachments/assets/93c7c913-d9d8-4f22-95c7-adf94f55f468)

Клонирование репозитория Grafana Stack для Docker с GitHub.

    sudo git clone https://github.com/skl256/grafana_stack_for_docker.git

![изображение](https://github.com/user-attachments/assets/8d0e763b-22e7-4937-81b8-8f1de119269f)

Заходим в папку grafana_stack_for_docker

    cd grafana_stack_for_docker

![изображение](https://github.com/user-attachments/assets/bdb42830-d931-4f39-b861-e1843423986e)

Cоздаем папки двумя разными способами

Создание каталога для конфигурации Grafana внутри общей директории в Docker Swarm.

     sudo mkdir -p /mnt/common_volume/swarm/grafana/config

Создание нескольких директорий для хранения данных конфигурации и информации для Grafana, Prometheus, Loki, Promtail.

     sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}


Выдаем права

     sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}

Создаем файл

     sudo touch /mnt/common_volume/grafana/grafana-config/grafana.ini

Копирование файлов

     sudo cp config/* /mnt/common_volume/swarm/grafana/config/

Переименовывание файла

     sudo mv grafana.yaml docker-compose.yaml

Запуск сервисов, описанных в docker-compose.yaml, в фоновом режиме.

     sudo docker compose up -d

![изображение](https://github.com/user-attachments/assets/9376e48f-1ef2-446b-97b5-0b0db1fbe5ce)

После запуска, нам нужно будет на время остановить docker - sudo docker compose stop

После остановки, продолжаем работать дальше

    sudo vi docker-compose.yaml

Затем в docker-compose нужно вставить node-exporter:

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

![изображение](https://github.com/user-attachments/assets/0a290be7-262f-42d0-9fcc-88e952f3b1c4)

Заходим в другую папку

    cd /mnt/common_volume/swarm/grafana/config

Либо

    cd grafana_stack_for_docker/config

Заходим в редактирование файласв

     sudo vi prometheus.yaml

Далее нужно исправить targets: на exporter:9100

![изображение](https://github.com/user-attachments/assets/c02f670f-f037-4784-90f9-2c38bd8583b4)

Сохраняем при помощи :wq! и снова запускаем Docker Compose

# => Установка Prometheus

Для установки и дальнейшей настройки, нам нужно зайти на сайт: http://localhost:3000

Логин и пароль: Admin

В меню выбираем вкладку Dashboards, нажимаем на кнопку "+ create dashnoard", затем на "+ add visualization" и после на "configure a new data source".
выбираем Prometheus, в connection нужно будет ввести: "http://prometheus:9090".
в authentication нужно будет поставить "Basic authentication" и ввести логин и пароль: admin, после этого нажимаем на "Save & test" и должно показать зеленую галочку
Authentication

![изображение](https://github.com/user-attachments/assets/e230491d-02c3-42b2-a557-ba03993ca545)

возвращаемся обратно в Dashboards и нажимаем снова Create Dashboard
нажимаем Import dashboard и в поле "Find and import dashboards" нужно будет ввести "1860", потом нужно будет выбрать Prometheus и импортировать dashboard

![изображение](https://github.com/user-attachments/assets/559efb90-a904-40c5-9ce8-28898e10cc54)

В конечном итоге, должно получиться вот так:

![изображение](https://github.com/user-attachments/assets/2fb8e36c-b079-42b3-99a2-8014f0ca2222)

# => Делаем VictoriaMetrics

Для начала зайдем в нужную папку

     cd grafana_stack_for_docker

Открываем docker-compose

     sudo vi docker-compose.yaml

После prometheus вставляем vmagent

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

Затем возвращаемся в localhost и переходим во вкладку connection, нажимаем на data sources и затем на prometheus. Нам нужно будет поменять в connection с "http://prometheus:9090" на "http://victoriametrics:8428" и поменять название на "Vika" 

![изображение](https://github.com/user-attachments/assets/b01e3029-9c6c-4ed7-a96c-6317a610da4b)

После возвращаемся в Dashboards и нажимаем на Add visualization, на этот раз выбираем VikMetric, в созданной Dashboard нам нужно будет выбрать вкладку "code"

![изображение](https://github.com/user-attachments/assets/d111c4b6-59a4-4806-8892-30e9055993c3)


После переходим в терминал и пишем эти команды:

     echo -e "# TYPE OILCOINT_metric1 gauge\nOILCOINT_metric1 0" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus

     curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric1'

![изображение](https://github.com/user-attachments/assets/d75b6297-863c-47f3-887c-9fa01baf1cf5)

Переходим на localhost:8428, нажимаем на vmui, копируем переменную OILCOINT_metric1 и вставляем в query

Нажимаем execute query

![изображение](https://github.com/user-attachments/assets/5d7fb63b-b01a-4f0b-bccb-7e1ea277edb9)

Копируем переменную OILCOINT_metric1 и вставляем в code и нажимаем run queries

![изображение](https://github.com/user-attachments/assets/f74c4807-a2cf-475c-b3f5-e97b3f8f73ec)


