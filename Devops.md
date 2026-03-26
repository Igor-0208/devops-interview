### **1. Как дебажить поломанные поды в Kubernetes?**

#### Ответ:

1. **Проверить статус пода:**
    
    `kubectl get pods -n <namespace>`
    
    Возможные статусы: `CrashLoopBackOff`, `ImagePullBackOff`, `Pending`, `Error`, `Completed` и т. д.
    
2. **Получить подробную информацию о поде:**
    
    `kubectl describe pod <pod_name>`
    
### **1. Как раскатывать ноды в Kubernetes?**

**Добавление ноды:**

1. **Установить зависимости:** Docker/Containerd, Kubelet, Kubeadm, Kubectl.
2. **Инициализация кластера (на мастере):**
    
    bash
    
    КопироватьРедактировать
    
    `kubeadm init`
    
3. **Получить токен для присоединения:**
    
    bash
    
    КопироватьРедактировать
    
    `kubeadm token create --print-join-command`
    
4. **Присоединить воркер-ноду:**  
    На новой ноде выполнить команду, полученную на предыдущем шаге:
    
    bash
    
    КопироватьРедактировать
    
    `kubeadm join <master-ip>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`
    
5. **Проверить статус:**
    
    bash
    
    КопироватьРедактировать
    
    `kubectl get nodes`
    

**Удаление ноды:**

bash

КопироватьРедактировать

`kubectl drain <node-name> --ignore-daemonsets --delete-local-data kubectl delete node <node-name>`

---

### ⚙️ **2. Как обращаться с сервисами?**

**Поиск сервисов:**

bash

КопироватьРедактировать

`kubectl get svc -n <namespace>`

**Создание сервиса:**

bash

КопироватьРедактировать

`kubectl expose deployment <deployment-name> --port=80 --target-port=8080 --type=ClusterIP`

**Доступ к сервису:**

bash

КопироватьРедактировать

`kubectl port-forward svc/<service-name> 8080:80 curl http://localhost:8080`

---

### 🔍 **3. Как проводить диагностику по выводу пода?**

1. **Посмотреть статус пода:**

bash

КопироватьРедактировать

`kubectl get pod <pod-name> -n <namespace> -o wide`

2. **Получить детали:**

bash

КопироватьРедактировать

`kubectl describe pod <pod-name> -n <namespace>`

3. **Логи контейнера:**

bash

КопироватьРедактировать

`kubectl logs <pod-name> -n <namespace> --tail=100`

4. **Зайти внутрь контейнера:**

bash

КопироватьРедактировать

`kubectl exec -it <pod-name> -n <namespace> -- /bin/sh`

5. **Проверить события:**

bash

КопироватьРедактировать

`kubectl get events --sort-by='.lastTimestamp'`