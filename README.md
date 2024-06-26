#  Курсовая работа на профессии "DevOps-инженер с нуля" - `Мурчин Артем`

## Доработка от 15.04.2024
## Файлы с кодом terraform:
main.tf - https://github.com/artmur1/15-hw/blob/main/main.tf.txt

meta.yml - https://github.com/artmur1/15-hw/blob/main/meta.yml.txt

В файле main.tf описано 6 виртуальных машин - bastion, vm1, vm2, zabbix, elk, kibana; target Group, backend Group, HTTP router, application load balancer, security groups, snapshot schedule.

## Файлы с кодом ansible:
playbook.yaml - https://github.com/artmur1/15-hw/blob/main/playbook.txt

hosts - https://github.com/artmur1/15-hw/blob/main/hosts.txt

В файле playbook.yaml описано развертывание nginx , zabbix, zabbix-agent, elk, kibana.

Содержание
==========
* [Задача](#Задача)
* [Инфраструктура](#Инфраструктура)
    * [Сайт](#Сайт)
    * [Мониторинг](#Мониторинг)
    * [Логи](#Логи)
    * [Сеть](#Сеть)
    * [Резервное копирование](#Резервное-копирование)
    * [Дополнительно](#Дополнительно)
* [Выполнение работы](#Выполнение-работы)
* [Критерии сдачи](#Критерии-сдачи)
* [Как правильно задавать вопросы дипломному руководителю](#Как-правильно-задавать-вопросы-дипломному-руководителю) 

---------
## Задача
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в [Yandex Cloud](https://cloud.yandex.com/).

## Инфраструктура
Для развёртки инфраструктуры используйте Terraform и Ansible. 

Параметры виртуальной машины (ВМ) подбирайте по потребностям сервисов, которые будут на ней работать. 

Ознакомьтесь со всеми пунктами из этой секции, не беритесь сразу выполнять задание, не дочитав до конца. Пункты взаимосвязаны и могут влиять друг на друга.

### Сайт
Создайте две ВМ в разных зонах, установите на них сервер nginx, если его там нет. ОС и содержимое ВМ должно быть идентичным, это будут наши веб-сервера.

Используйте набор статичных файлов для сайта. Можно переиспользовать сайт из домашнего задания.

Создайте [Target Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/target-group), включите в неё две созданных ВМ.

Создайте [Backend Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/backend-group), настройте backends на target group, ранее созданную. Настройте healthcheck на корень (/) и порт 80, протокол HTTP.

Создайте [HTTP router](https://cloud.yandex.com/docs/application-load-balancer/concepts/http-router). Путь укажите — /, backend group — созданную ранее.

Создайте [Application load balancer](https://cloud.yandex.com/en/docs/application-load-balancer/) для распределения трафика на веб-сервера, созданные ранее. Укажите HTTP router, созданный ранее, задайте listener тип auto, порт 80.

Протестируйте сайт
`curl -v <публичный IP балансера>:80` 

### Решение - Сайт

Создал vm1-u22-04 в зоне ru-central1-b и vm2-u22-04 в зоне ru-central1-d. Обе виртмашины работают на Ubuntu 22.04 LTS.

Адреса вм:
    
vm1-u22-04 - http://84.201.141.114/
    
vm2-u22-04 - http://158.160.154.87/

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-1.png)

Создал manage-ubuntu22-04 для управления. Установил ansible. Написал плейбук для установки nginx на виртмашины.

Плейбук для установки nginx на виртмашины - https://github.com/artmur1/15-hw/blob/main/15-1-1-4-playbook.txt

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-1-3-ansible.png)

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-1-2-ansible.png)

Создал Target Group, включил в неё две созданных ВМ.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-2-1-target_group.png)

Создал Backend Group, настроил backends на target group, ранее созданную. Настройте healthcheck на корень (/) и порт 80, протокол HTTP.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-3-backend-group.png)

Создал HTTP router. Путь указал — /, backend group — указал созданную ранее.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-4-http-router.png)

Создал Application load balancer для распределения трафика на веб-сервера, созданные ранее. Указал HTTP router, созданный ранее, задал listener тип auto, порт 80. 

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-5-1-application-load-balancer.png)

Результат теста сайта. Указал публичный IP балансера

    curl -v 158.160.155.208:80

![alt text](https://github.com/artmur1/15-hw/blob/main/15-1-5-2-application-load-balancer.png)

### Мониторинг
Создайте ВМ, разверните на ней Zabbix. На каждую ВМ установите Zabbix Agent, настройте агенты на отправление метрик в Zabbix. 

Настройте дешборды с отображением метрик, минимальный набор — по принципу USE (Utilization, Saturation, Errors) для CPU, RAM, диски, сеть, http запросов к веб-серверам. Добавьте необходимые tresholds на соответствующие графики.

### Решение - Мониторинг

Создал ВМ, развернул на ней Zabbix. На каждую ВМ установите Zabbix Agent, настройте агенты на отправление метрик в Zabbix.

Zabbix сервер находится по адресу http://51.250.98.165/zabbix/index.php

![alt text](https://github.com/artmur1/15-hw/blob/main/15-2-1.png)

Дешборды с отображением метрик.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-2-2.png)

### Логи
Cоздайте ВМ, разверните на ней Elasticsearch. Установите filebeat в ВМ к веб-серверам, настройте на отправку access.log, error.log nginx в Elasticsearch.

Создайте ВМ, разверните на ней Kibana, сконфигурируйте соединение с Elasticsearch.

### Решение - Логи

Cоздал ВМ, развернул на ней Elasticsearch.

Elasticsearch находится по адресу http://84.201.178.26:9200/

![alt text](https://github.com/artmur1/15-hw/blob/main/15-3-1.png)

Установил Vector на обе ВМ: vm1-u22-04, vm2-u22-04.

Конфиг Vector - https://github.com/artmur1/15-hw/blob/main/15-3-5-Vector.txt

![alt text](https://github.com/artmur1/15-hw/blob/main/15-3-1-2.png)

Настроил на отправку access.log, error.log nginx в Elasticsearch.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-3-3.png)

Создал ВМ, развернул на ней Kibana, сконфигурировал соединение с Elasticsearch.

Kibana находится по адресу http://62.84.123.190:5601/

![alt text](https://github.com/artmur1/15-hw/blob/main/15-3-2.png)

![alt text](https://github.com/artmur1/15-hw/blob/main/15-3-4.png)

### Сеть
Разверните один VPC. Сервера web, Elasticsearch поместите в приватные подсети. Сервера Zabbix, Kibana, application load balancer определите в публичную подсеть.

Настройте [Security Groups](https://cloud.yandex.com/docs/vpc/concepts/security-groups) соответствующих сервисов на входящий трафик только к нужным портам.

Настройте ВМ с публичным адресом, в которой будет открыт только один порт — ssh. Настройте все security groups на разрешение входящего ssh из этой security group. Эта вм будет реализовывать концепцию bastion host. Потом можно будет подключаться по ssh ко всем хостам через этот хост.

### Решение - Сеть

Развернул сеть VPC. Создал приватную и публичную подсети. Прописал порты.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-4-1.png)

![alt text](https://github.com/artmur1/15-hw/blob/main/15-4-2.png)

![alt text](https://github.com/artmur1/15-hw/blob/main/15-4-3.png)

![alt text](https://github.com/artmur1/15-hw/blob/main/15-4-4.png)

![alt text](https://github.com/artmur1/15-hw/blob/main/15-4-5.png)

### Резервное копирование
Создайте snapshot дисков всех ВМ. Ограничьте время жизни snaphot в неделю. Сами snaphot настройте на ежедневное копирование.

### Решение - Резервное копирование

Создал snapshot дисков всех ВМ.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-5-1.png)

Ограничил время жизни snaphot в неделю. Сами snaphot настроил на ежедневное копирование.

![alt text](https://github.com/artmur1/15-hw/blob/main/15-5-2.png)

### Дополнительно
Не входит в минимальные требования. 

1. Для Zabbix можно реализовать разделение компонент - frontend, server, database. Frontend отдельной ВМ поместите в публичную подсеть, назначте публичный IP. Server поместите в приватную подсеть, настройте security group на разрешение трафика между frontend и server. Для Database используйте [Yandex Managed Service for PostgreSQL](https://cloud.yandex.com/en-ru/services/managed-postgresql). Разверните кластер из двух нод с автоматическим failover.
2. Вместо конкретных ВМ, которые входят в target group, можно создать [Instance Group](https://cloud.yandex.com/en/docs/compute/concepts/instance-groups/), для которой настройте следующие правила автоматического горизонтального масштабирования: минимальное количество ВМ на зону — 1, максимальный размер группы — 3.
3. В Elasticsearch добавьте мониторинг логов самого себя, Kibana, Zabbix, через filebeat. Можно использовать logstash тоже.
4. Воспользуйтесь Yandex Certificate Manager, выпустите сертификат для сайта, если есть доменное имя. Перенастройте работу балансера на HTTPS, при этом нацелен он будет на HTTP веб-серверов.

## Выполнение работы
На этом этапе вы непосредственно выполняете работу. При этом вы можете консультироваться с руководителем по поводу вопросов, требующих уточнения.

⚠️ В случае недоступности ресурсов Elastic для скачивания рекомендуется разворачивать сервисы с помощью docker контейнеров, основанных на официальных образах.

**Важно**: Ещё можно задавать вопросы по поводу того, как реализовать ту или иную функциональность. И руководитель определяет, правильно вы её реализовали или нет. Любые вопросы, которые не освещены в этом документе, стоит уточнять у руководителя. Если его требования и указания расходятся с указанными в этом документе, то приоритетны требования и указания руководителя.

## Критерии сдачи
1. Инфраструктура отвечает минимальным требованиям, описанным в [Задаче](#Задача).
2. Предоставлен доступ ко всем ресурсам, у которых предполагается веб-страница (сайт, Kibana, Zabbix).
3. Для ресурсов, к которым предоставить доступ проблематично, предоставлены скриншоты, команды, stdout, stderr, подтверждающие работу ресурса.
4. Работа оформлена в отдельном репозитории в GitHub или в [Google Docs](https://docs.google.com/), разрешён доступ по ссылке. 
5. Код размещён в репозитории в GitHub.
6. Работа оформлена так, чтобы были понятны ваши решения и компромиссы. 
7. Если использованы дополнительные репозитории, доступ к ним открыт. 

## Как правильно задавать вопросы дипломному руководителю
Что поможет решить большинство частых проблем:
1. Попробовать найти ответ сначала самостоятельно в интернете или в материалах курса и только после этого спрашивать у дипломного руководителя. Навык поиска ответов пригодится вам в профессиональной деятельности.
2. Если вопросов больше одного, присылайте их в виде нумерованного списка. Так дипломному руководителю будет проще отвечать на каждый из них.
3. При необходимости прикрепите к вопросу скриншоты и стрелочкой покажите, где не получается. Программу для этого можно скачать [здесь](https://app.prntscr.com/ru/).

Что может стать источником проблем:
1. Вопросы вида «Ничего не работает. Не запускается. Всё сломалось». Дипломный руководитель не сможет ответить на такой вопрос без дополнительных уточнений. Цените своё время и время других.
2. Откладывание выполнения дипломной работы на последний момент.


