1. Как дебажить поломанные поды в Kubernetes?

- Ответ:  
Проверить статус пода:  
> kubectl get pods -n <namespace>  

Возможные статусы: CrashLoopBackOff, ImagePullBackOff, Pending, Error, Completed и т. д.  
Получить подробную информацию о поде:  

> kubectl describe pod <pod_name>
