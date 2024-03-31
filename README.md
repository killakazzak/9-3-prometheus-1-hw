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
systemctl statu node-exporter
```

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

---

### Задание 4*
Установите Grafana.

#### Требования к результату
- [ ] Прикрепите к файлу README.md скриншот левого нижнего угла интерфейса, чтобы при наведении на иконку пользователя были видны ваши ФИО

### Решение Задание 4*
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
