# Домашнее задание к занятию "`Система мониторинга Prometheus`" - `Тен Денис`

### Задание 1*
Установите Prometheus.

#### Процесс выполнения
1. Выполняя задание, сверяйтесь с процессом, отражённым в записи лекции
2. Создайте пользователя prometheus
3. Скачайте prometheus и в соответствии с лекцией разместите файлы в целевые директории
4. Создайте сервис как показано на уроке
5. Проверьте что prometheus запускается, останавливается, перезапускается и отображает статус с помощью systemctl

#### Требования к результату
- [ ] Прикрепите к файлу README.md скриншот systemctl status prometheus, где будет написано: prometheus.service — Prometheus Service Netology Lesson 9.4 — [Ваши ФИО]

### Решение Задание 1*

Создаём пользователя Prometheus
```
sudo useradd --no-create-home --shell /bin/false prometheus
```
Найдём последнюю версию Prometheus на :GitHub
```
wget https://github.com/prometheus/prometheus/releases/download/v2.51.1/prometheus-2.51.1.linux-386.tar.gz
```

Извлекаем архив и скопируем файлы в необходимые директории:
```
tar xfvz prometheus-2.51.1.linux-386.tar.gz
cd prometheus-2.51.1.linux-386
mkdir /etc/prometheus
mkdir /var/lib/prometheus
cp promtool prometheus /usr/local/bin/
cp -R console_libraries/ /etc/prometheus/
cp prometheus.yml /etc/prometheus/
chown -R prometheus:prometheus /etc/prometheus/ /var/lib/prometheus/
chown -R prometheus:prometheus /var/lib/prometheus
chown  prometheus:prometheus /usr/local/bin/promtool /usr/local/bin/prometheus
```
Проверяем
```
/usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries
```
![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/9bd4edcd-1a66-4513-bf3b-5afee96acefd)

Создаем сервис
```
vim /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus Service Netology Lesson 9.4
After=network.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
ExecReload=/bin/kill -HUP $MAINPID Restart=on-failure
[Install]
WantedBy=multi-user.target
```
```
systemctl daemon-reload
systemctl enable --now prometheus.service
```

Проверяем
```
systemctl status prometheus
```
![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/44c1935b-326c-4a44-9dcb-ab56f10064c4)


### Задание 2*
Установите Node Exporter.

#### Процесс выполнения
1. Выполняя ДЗ сверяйтесь с процессом отражённым в записи лекции.
3. Скачайте node exporter приведённый в презентации и в соответствии с лекцией разместите файлы в целевые директории
4. Создайте сервис для как показано на уроке
5. Проверьте что node exporter запускается, останавливается, перезапускается и отображает статус с помощью systemctl

#### Требования к результату
- [ ] Прикрепите к файлу README.md скриншот systemctl status node-exporter, где будет написано: node-exporter.service — Node Exporter Netology Lesson 9.4 — [Ваши ФИО]

### Решение Задание 2*

Скачиваем и извлекаем архив последней версии node_exporter
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-386.tar.gz
Извлекаем архив и скопируем файлы в необходимые директории:
tar xvfz node_exporter-1.7.0.linux-386.tar.gz
cd node_exporter-1.7.0.linux-386/
mkdir /etc/prometheus/node-exporter
cp ./* /etc/prometheus/node-exporter/
chown -R prometheus:prometheus /etc/prometheus/node-exporter/
```
Создаем сервис node-exporter.service
```
vim /etc/systemd/system/node-exporter.service
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
Проверяем службу

```
systemctl enable --now  node-exporter
systemctl status node-exporter
```
Редактируем конфигурацию Prometheus
```
vim /etc/prometheus/prometheus.yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
- job_name: "prometheus"
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
  static_configs:
    - targets:  ['localhost:9090','localhost:9100']
```
Перезапускаем Prometheus
```
systemctl restart prometheus
```

![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/1551ffb4-b098-47b5-8e44-d5cc29748924)

![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/48772da3-4ef3-49be-b5ca-5cd32d996240)

---

### Задание 3*
Подключите Node Exporter к серверу Prometheus.

#### Процесс выполнения
1. Выполняя ДЗ сверяйтесь с процессом отражённым в записи лекции.
2. Отредактируйте prometheus.yaml, добавив в массив таргетов установленный в задании 2 node exporter
3. Перезапустите prometheus
4. Проверьте что он запустился

#### Требования к результату
- [ ] Прикрепите к файлу README.md скриншот конфигурации из интерфейса Prometheus вкладки Status > Configuration
- [ ] Прикрепите к файлу README.md скриншот из интерфейса Prometheus вкладки Status > Targets, чтобы было видно минимум два эндпоинта

---
## Дополнительные задания со звёздочкой*
Эти задания дополнительные. Их можно не выполнять. Это не повлияет на зачёт. Вы можете их выполнить, если хотите глубже разобраться в материале.

### Решение Задание 3*
![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/8aa1856e-72d2-4cbf-a4ea-755d09369fc1)
![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/e51b29ea-8e71-4191-a650-b0e92a1aa4ec)
![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/f5a6ace6-ad6c-435e-bb47-784e170f762f)


---

### Задание 4*
Установите Grafana.

#### Требования к результату
- [ ] Прикрепите к файлу README.md скриншот левого нижнего угла интерфейса, чтобы при наведении на иконку пользователя были видны ваши ФИО

### Решение Задание 4*

Скачиваем и устанавливаем последнюю версию Grafana
```
wget https://dl.grafana.com/oss/release/grafana_10.4.1_amd64.deb
dpkg -i grafana_10.4.1_amd64.deb
```
![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/b1da6751-1eca-47fa-81e6-19d117a1bc01)

Проверяем установку и работу службы
```
systemctl enable --now grafana-server.service
systemctl status grafana-server.service
```
![image](https://github.com/killakazzak/hw-prometheus-01/assets/32342205/50d11910-3641-4c0f-b79f-685909913a45)



---

### Задание 5*
Интегрируйте Grafana и Prometheus.

#### Требования к результату
- [ ] Прикрепите к файлу README.md скриншот дашборда (ID:11074) с поступающими туда данными из Node Exporter

### Решение Задание 5*

## Критерии оценки
1. Выполнено минимум 3 обязательных задания
2. Прикреплены требуемые скриншоты
3. Задание оформлено в шаблоне с решением и опубликовано на GitHub
