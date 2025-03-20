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

>kubectl drain <node-name> --ignore-daemonsets --delete-local-data

>kubectl delete node <node-name>

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
