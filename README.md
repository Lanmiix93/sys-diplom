
#  Дипломная работа по профессии «Системный администратор»

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
Ключевая задача — разработать отказоустойчивую инфраструктуру для сайта, включающую мониторинг, сбор логов и резервное копирование основных данных. Инфраструктура должна размещаться в [Yandex Cloud](https://cloud.yandex.com/) и отвечать минимальным стандартам безопасности: запрещается выкладывать токен от облака в git. Используйте [инструкцию](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart#get-credentials).

**Перед началом работы над дипломным заданием изучите [Инструкция по экономии облачных ресурсов](https://github.com/netology-code/devops-materials/blob/master/cloudwork.MD).**

## Инфраструктура
Для развёртки инфраструктуры используйте Terraform и Ansible.  

Не используйте для ansible inventory ip-адреса! Вместо этого используйте fqdn имена виртуальных машин в зоне ".ru-central1.internal". Пример: example.ru-central1.internal  

Важно: используйте по-возможности **минимальные конфигурации ВМ**:2 ядра 20% Intel ice lake, 2-4Гб памяти, 10hdd, прерываемая. 

**Так как прерываемая ВМ проработает не больше 24ч, перед сдачей работы на проверку дипломному руководителю сделайте ваши ВМ постоянно работающими.**

Ознакомьтесь со всеми пунктами из этой секции, не беритесь сразу выполнять задание, не дочитав до конца. Пункты взаимосвязаны и могут влиять друг на друга.

Поднимаем инфраструктуру через Terraform:

![Скриншот-1](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Terraform.PNG)

![Скриншот-2](https://github.com/Lanmiix93/sys-diplom/blob/main/img/VMs-YC.PNG)

### Сайт
Создайте две ВМ в разных зонах, установите на них сервер nginx, если его там нет. ОС и содержимое ВМ должно быть идентичным, это будут наши веб-сервера.

Используйте набор статичных файлов для сайта. Можно переиспользовать сайт из домашнего задания.

Создайте [Target Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/target-group), включите в неё две созданных ВМ.

Создайте [Backend Group](https://cloud.yandex.com/docs/application-load-balancer/concepts/backend-group), настройте backends на target group, ранее созданную. Настройте healthcheck на корень (/) и порт 80, протокол HTTP.

Создайте [HTTP router](https://cloud.yandex.com/docs/application-load-balancer/concepts/http-router). Путь укажите — /, backend group — созданную ранее.

Создайте [Application load balancer](https://cloud.yandex.com/en/docs/application-load-balancer/) для распределения трафика на веб-сервера, созданные ранее. Укажите HTTP router, созданный ранее, задайте listener тип auto, порт 80.

Протестируйте сайт
`curl -v <публичный IP балансера>:80` 

Target Group:
![Скриншот-3](https://github.com/Lanmiix93/sys-diplom/blob/main/img/target-group.PNG)

Backend Group:
![Скриншот-4](https://github.com/Lanmiix93/sys-diplom/blob/main/img/backend-group.PNG)

HTTP Router:
![Скриншот-5](https://github.com/Lanmiix93/sys-diplom/blob/main/img/http-router.PNG)

Application load Balancer:
![Скриншот-6](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Load-Balancer.PNG)

Curl сайта (Доступен по адресу 158.160.130.232:80):
![Скриншот-7](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Webserver_Curl.PNG)

### Мониторинг
Создайте ВМ, разверните на ней Zabbix. На каждую ВМ установите Zabbix Agent, настройте агенты на отправление метрик в Zabbix. 

Настройте дешборды с отображением метрик, минимальный набор — по принципу USE (Utilization, Saturation, Errors) для CPU, RAM, диски, сеть, http запросов к веб-серверам. Добавьте необходимые tresholds на соответствующие графики.

С помощью ansible развернута Prometheus и Grafana (Доступен по адресу 158.160.60.94:3000 admin:admin) : 
![Скриншот-8](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Monitoring-1.PNG)
![Скриншот-9](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Monitoring-2.PNG)
![Скриншот-10](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Monitoring-3.PNG)

### Логи
Cоздайте ВМ, разверните на ней Elasticsearch. Установите filebeat в ВМ к веб-серверам, настройте на отправку access.log, error.log nginx в Elasticsearch.

Создайте ВМ, разверните на ней Kibana, сконфигурируйте соединение с Elasticsearch.

С помощью ansible развернут elasticsearch и kibana. Указаны логи Nginx error и Access (Доступен по адресу 158.160.126.196:5601) :
![Скриншот-11](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Elastic.PNG)
![Скриншот-12](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Elastic2.PNG)

### Сеть
Разверните один VPC. Сервера web, Elasticsearch поместите в приватные подсети. Сервера Zabbix, Kibana, application load balancer определите в публичную подсеть.

Настройте [Security Groups](https://cloud.yandex.com/docs/vpc/concepts/security-groups) соответствующих сервисов на входящий трафик только к нужным портам.

Настройте ВМ с публичным адресом, в которой будет открыт только один порт — ssh.  Эта вм будет реализовывать концепцию  [bastion host]( https://cloud.yandex.ru/docs/tutorials/routing/bastion) . Синоним "bastion host" - "Jump host". Подключение  ansible к серверам web и Elasticsearch через данный bastion host можно сделать с помощью  [ProxyCommand](https://docs.ansible.com/ansible/latest/network/user_guide/network_debug_troubleshooting.html#network-delegate-to-vs-proxycommand) . Допускается установка и запуск ansible непосредственно на bastion host.(Этот вариант легче в настройке)

Security Group:
![Скриншот-13](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Security-Group.PNG)

Subnet:
![Скриншот-14](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Subnet.PNG)

Bastion Host. Пинги до всех ВМ через бастион хост:
![Скриншот-15](https://github.com/Lanmiix93/sys-diplom/blob/main/img/All_Hosts_Ping.PNG)

### Резервное копирование
Создайте snapshot дисков всех ВМ. Ограничьте время жизни snaphot в неделю. Сами snaphot настройте на ежедневное копирование.

![Скриншот-16](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Snapshot-1.PNG)
![Скриншот-17](https://github.com/Lanmiix93/sys-diplom/blob/main/img/Snapshot-2.PNG)
