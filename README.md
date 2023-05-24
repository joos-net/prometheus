# Домашнее задание к занятию "`Система мониторинга Prometheus`"
# `Островский Евгений`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

Установите Prometheus.

Создаем пользователя
```
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
Качаем последнюю версию
```
wget https://github.com/prometheus/prometheus/releases/download/v2.40.1/prometheus-2.40.1.linux-amd64.tar.gz
tar -xvf prometheus-2.40.1.linux-amd64.tar.gz
```
Создаем и копируем необходимые файлы и директории
```
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
cd prometheus-2.40.1.linux-amd64
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```
Передаем права
```
chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus 
chown prometheus:prometheus /usr/local/bin/prometheus 
chown prometheus:prometheus /usr/local/bin/promtool
```
Создаем сервис
```
nano /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus Service Netology Lesson 9.4 Ostrowskij E.
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Restart=always
Type=simple
ExecReload=/bin/kill -HUP $MAINPID Restart=on-failure
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090
[Install]
WantedBy=multi-user.target
```
Передаем права
```
chown -R prometheus:prometheus /var/lib/prometheus
```
Автозапуск и запуск
```
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

### Требования к результату

Прикрепите к файлу README.md скриншот systemctl status prometheus, где будет написано: prometheus.service — Prometheus Service Netology Lesson 9.4 — [Ваши ФИО]

![Task1](https://github.com/joos-net/prometheus/blob/main/graph1.png)


---

### Задание 2

Установите Node Exporter.

Качаем и извлекаем
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz 
tar xvfz node_exporter-*.*-amd64.tar.gz
cd node_exporter-*.*-amd64
```

Копируем в папку Prometheus и передаем права
```
mkdir /etc/prometheus/node-exporter
cp ./* /etc/prometheus/node-exporter
chown -R prometheus:prometheus /etc/prometheus/node-exporter/
```
Создаем сервис
```
nano /etc/systemd/system/node-exporter.service
```
```
[Unit]
Description=Node Exporter Lesson 9.4
After=network.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/etc/prometheus/node-exporter/node_exporter
[Install]
WantedBy=multi-user.target
```
Прописываем автозапуск и запускаем
```
sudo systemctl enable node-exporter
sudo systemctl start node-exporter
sudo systemctl status node-exporter
```

### Требования к результату

Прикрепите к файлу README.md скриншот systemctl status node-exporter, где будет написано: node-exporter.service — Node Exporter Netology Lesson 9.4 — [Ваши ФИО]

![Task2](https://github.com/joos-net/prometheus/blob/main/graph2.png)

---

### Задание 3

Подключите Node Exporter к серверу Prometheus.

Редактируем
```
nano /etc/prometheus/prometheus.yml
```
```
static_configs:
— targets: ['localhost:9090', 'localhost:9100']
```
Перезапускаем Prometheus
```
systemctl restart prometheus
```

### Требования к результату

1. Прикрепите к файлу README.md скриншот конфигурации из интерфейса Prometheus вкладки Status > Configuration
2. Прикрепите к файлу README.md скриншот из интерфейса Prometheus вкладки Status > Targets, чтобы было видно минимум два эндпоинта

![Task3](https://github.com/joos-net/prometheus/blob/main/graph30.png)
![Task31](https://github.com/joos-net/prometheus/blob/main/graph31.png)


---
## Дополнительные задания (со звездочкой*)

Эти задания дополнительные (не обязательные к выполнению) и никак не повлияют на получение вами зачета по этому домашнему заданию. Вы можете их выполнить, если хотите глубже и/или шире разобраться в материале.

### Задание 4

Установите Grafana.

Качаем и устанавилваем
```
wget https://dl.grafana.com/oss/release/grafana_9.2.4_amd64.deb
dpkg -i grafana_9.2.4_amd64.deb
```
Включаем автозапуск и запускаем
```
systemctl enable grafana-server
systemctl start grafana-server
systemctl status grafana-server
```

### Требования к результату

Прикрепите к файлу README.md скриншот левого нижнего угла интерфейса, чтобы при наведении на иконку пользователя были видны ваши ФИО

![Task4](https://github.com/joos-net/prometheus/blob/main/graph4.png)

### Задание 5

Интегрируйте Grafana и Prometheus.

1. Перейдите в раздел Configuration > Data Sources и нажмите на Add data sourcе 
2. В появившемся списке выберите Prometheus:
``` 
http://localhost:9090
```
3. Нажмите кнопку Save & Test На сайте grafana.com найдите нужный дашборд и скопируйте его ID
4. Перейдите в раздел [+] > Import и в поле Import via grafana.com внесите скопированный ID или ссылку на дашборд 
5. В выпадающем списке VictoriaMetrics выберите Prometheus Нажмите кнопку Import

### Требования к результату

Прикрепите к файлу README.md скриншот дашборда (ID:11074) с поступающими туда данными из Node Exporter

![Task5](https://github.com/joos-net/prometheus/blob/main/graph5.png)
