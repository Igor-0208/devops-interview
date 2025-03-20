### Kubernetes K8s
## 1. Как дебажить поломанные поды в Kubernetes?

- Ответ:  
Проверить статус пода:  
> kubectl get pods -n <namespace>  

Возможные статусы: **CrashLoopBackOff, ImagePullBackOff, Pending, Error, Completed** и т. д.  
Получить подробную информацию о поде:  

> kubectl describe pod <pod_name>

## 2. Как раскатывать ноды в Kubernetes?  
- Ответ:  
Добавление ноды:  

Установить зависимости: Docker/Containerd, Kubelet, Kubeadm, Kubectl.  
Инициализация кластера (на мастере):  

> kubeadm init

Получить токен для присоединения:  

> kubeadm token create --print-join-command

Присоединить воркер-ноду:  
На новой ноде выполнить команду, полученную на предыдущем шаге:  

> kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

Проверить статус:  
>kubectl get nodes

Удаление ноды:  

> kubectl drain <node-name> --ignore-daemonsets --delete-local-data

> kubectl delete node <node-name>

## ⚙️ 3. Как обращаться с сервисами?  
- Ответ:  
Поиск сервисов:  

> kubectl get svc -n <namespace>  

Создание сервиса:  

> kubectl expose deployment <deployment-name> --port=80 --target-port=8080 --type=ClusterIP

Доступ к сервису:  

> kubectl port-forward svc/<service-name> 8080:80  

> curl http://localhost:8080  

## 🔍 4. Как проводить диагностику по выводу пода?  
Посмотреть статус пода:  

> kubectl get pod <pod-name> -n <namespace> -o wide  

Получить детали:  

> kubectl describe pod <pod-name> -n <namespace>

Логи контейнера:  

> kubectl logs <pod-name> -n <namespace> --tail=100

Зайти внутрь контейнера:  

> kubectl exec -it <pod-name> -n <namespace> -- /bin/sh

Проверить события:  

> kubectl get events --sort-by='.lastTimestamp'  

### Docker

## 1. Команды Dockerfile

RUN — выполнение команды при сборке.  
CMD — команда по умолчанию при запуске контейнера.  

## 2. Мультистейдж сборки  
Используется несколько From для уменьшения размера образа  
Плюсы: меньше итоговый образ, не тащим инструменты сборки в прод.  

## 3. Оптимизация кеша (докидывание слоев)  
Правило: редко изменяемые слои ставим раньше, часто — позже.  

Dockerfile  

# Сначала зависимости (реже меняются)  
> COPY package.json package-lock.json ./  
> RUN npm ci  

# Потом исходный код (чаще меняется)  
> COPY . .  

Так при изменении кода пересобирается только финальная часть, кеш для зависимостей сохраняется. 🚀

### Docker-compose 

Сборка:  

> docker-compose build  

Запуск:  

> docker-compose up -d  

Пересборка без кеша:  

> docker-compose build --no-cache  
  
### ⚓ Helm для DevOps: основа и практика 🚀  
## 🏗️ 1. Из чего состоят чарты?  
Helm-чарт — это набор манифестов Kubernetes, шаблонизированных для удобства развертывания. Основные элементы чарта:  

📦 Chart.yaml — описание чарта (имя, версия, описание).  
📦 values.yaml — значения по умолчанию для шаблонов.  
📦 templates/ — сами манифесты K8s, которые используют переменные из values.  
📦 _helpers.tpl — шаблонные функции, например для именования ресурсов.  
📦 README.md — описание чарта и примеры использования (если повезёт).  
Пример структуры чарта:  

> my-chart/  
├── Chart.yaml  
├── values.yaml  
└── templates/  
    ├── deployment.yaml  
    ├── service.yaml  
    └── _helpers.tpl  

## ⚙️ 2. Добавление сервисов по образу и подобию  

Пример **service.yaml**:  

> apiVersion: v1  
kind: Service  
metadata:  
  name: {{ include "my-chart.fullname" . }}  
spec:  
  ports:  
    - port: {{ .Values.service.port }}  
      targetPort: {{ .Values.service.targetPort }}  
  selector:  
    app: {{ .Values.appName }}  

В **values.yaml** добавляем:  


> appName: my-app  
service:  
  port: 80  
  targetPort: 8080  
 
Применить чарт:  

> helm upgrade --install my-app ./my-chart

## 🧠 3. Как сделать values проще и понятнее?  
Группировать параметры по логическим блокам:  

> replicaCount: 3  
image:  
  repository: nginx  
  tag: latest  
  pullPolicy: IfNotPresent  
service:  
  type: ClusterIP  
  port: 80  

** Добавить комментарии, примеры и дефолтные значения.  **
** Минимизировать вложенность.  **
 
## 🔍 4. Рендеринг манифестов (dry-run)  
Проверить, что отрендерится:  

> helm template my-app ./my-chart --values values.yaml

## 🚀 5. Качаем апгрейды из консоли  
Установка/обновление:  

> helm upgrade --install my-app ./my-chart -f values.yaml

Откат:  

> helm rollback my-app 1

Удаление:  

> helm uninstall my-app  

Проверка состояния релиза:

> helm list
> helm status my-app

### Terraform

## 🏗️ 1. Как не поломать и не залочить стейт?  
❌ Проблема: если кто-то уже работает с инфраструктурой, можно получить конфликт блокировки.  
✅ Решение:  
Использовать remote backend (например, S3 с DynamoDB для блокировок):  
Проверить блокировку перед началом работы:  

> terraform state list  

Если стейт залочен (и ты уверен, что можно сбросить):  

> terraform force-unlock <LOCK_ID>  

## 🔄 2. Как перекатывать стейт?  
Цель: перенести стейт из одного backend-а в другой.  

Инициализировать старый стейт:  

> terraform init

Сделать бэкап:  

> cp terraform.tfstate terraform.tfstate.backup

Изменить backend в конфиге и выполнить миграцию:  

> terraform init -migrate-state

Проверить корректность:  

> terraform state list


## 📥 3. Как импортировать сущности в стейт?

Сценарий: ресурс уже создан руками, но Terraform про него не знает.  

Импортируй существующую сущность:  

> terraform import aws_instance.my_instance i-1234567890abcdef0

Проверь стейт:  

> terraform state show aws_instance.my_instance

Проверь, что план чистый:  

> terraform plan

## Быстрые команды:
Инициализация:  

> terraform init

Проверка плана:  

> terraform plan

Применение изменений:  

> terraform apply

Просмотр текущего стейта:  

> terraform state list

### Сети

## ⚙️ 1. Дебаг MTU (Maximum Transmission Unit)  
MTU — это максимальный размер пакета, который может передаваться по сети. Если MTU настроен неправильно, возможны проблемы с фрагментацией пакетов и потеря соединения.  

Проверка MTU на интерфейсе:  

> ip link show eth0

Здесь MTU = 1500.  

Тестирование MTU с помощью ping:  

> ping -c 4 -M do -s 1472 <IP-адрес>  

-M do — запрещает фрагментацию.  
-s — задаёт размер пакета (1472 = 1500 – 20 (IP) – 8 (ICMP)).  
Если MTU занижен, пакеты не пройдут.  

Изменение MTU:  

> ip link set eth0 mtu 1400

## 📶 2. Понимание пингов и трассировки
Ping:  

> ping -c 4 google.com

Вывод покажет задержку (time), потери пакетов и TTL (Time To Live).  

Traceroute:  
Показывает путь пакета через узлы сети:  

> traceroute google.com

Если где-то в цепочке долгое время отклика или * * * — есть проблемы с этим узлом.  

В Kubernetes:  
Проверь доступность сервиса:  

> kubectl exec -it <pod> -- ping <service-name>

## 🌐 3. Работа с DNS
Проверка резолва DNS:  

> nslookup google.com

>dig google.com

Проверка DNS-сервера в системе: 

> cat /etc/resolv.conf

Пример вывода:  

>nameserver 8.8.8.8

Очистка кеша DNS:

> systemctl restart systemd-resolved

## 📦 4. DNS в Kubernetes  
DNS-сервис: kube-dns (или CoreDNS).  
Имя сервиса:  

> kubectl get svc kube-dns -n kube-system

Проверка резолва из пода:  

> kubectl exec -it <pod> -- nslookup my-service.default.svc.cluster.local

Форматы DNS-имен в Кубере:  

Полное имя: my-service.default.svc.cluster.local  
Краткое имя: my-service (если в том же namespace).  
Диагностика:  

> kubectl logs -n kube-system -l k8s-app=kube-dns

## 🚦 Быстрые команды:  
Проверить доступность порта:  

> nc -zv <IP> <порт>
> nmap -Pn <IP> <порт>

Проверить маршрут:  

> ip route show

Проверить открытые порты:
> lsof -tulnp

> ss -tuln
