# Домашнее задание к занятию "12.2 Команды для работы с Kubernetes"
Кластер — это сложная система, с которой крайне редко работает один человек. Квалифицированный devops умеет наладить работу всей команды, занимающейся каким-либо сервисом.
После знакомства с кластером вас попросили выдать доступ нескольким разработчикам. Помимо этого требуется служебный аккаунт для просмотра логов.

## Задание 1: Запуск пода из образа в деплойменте
Для начала следует разобраться с прямым запуском приложений из консоли. Такой подход поможет быстро развернуть инструменты отладки в кластере. Требуется запустить деплоймент на основе образа из hello world уже через deployment. Сразу стоит запустить 2 копии приложения (replicas=2). 

Требования:
 * пример из hello world запущен в качестве deployment
 * количество реплик в deployment установлено в 2
```
ubuntu@devkub-vm:~$ cat hello-world-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment
  labels:
    app: hello-world
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: k8s.gcr.io/echoserver:1.4
        ports:
        - containerPort: 80
```
 * наличие deployment можно проверить командой kubectl get deployment
```
ubuntu@devkub-vm:~$ kubectl apply -f hello-world-deployment.yaml 
deployment.apps/hello-world-deployment configured
ubuntu@devkub-vm:~$ kubectl get deployment
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
hello-world-deployment   2/2     2            2           22m
```
 * наличие подов можно проверить командой kubectl get pods
```
ubuntu@devkub-vm:~$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
hello-world-deployment-d7ff4699c-7zzmx   1/1     Running   0          2m16s
hello-world-deployment-d7ff4699c-8ppr9   1/1     Running   0          2m16s
```


## Задание 2: Просмотр логов для разработки
Разработчикам крайне важно получать обратную связь от штатно работающего приложения и, еще важнее, об ошибках в его работе. 
Требуется создать пользователя и выдать ему доступ на чтение конфигурации и логов подов в app-namespace.

Требования: 
 * создан новый токен доступа для пользователя
 * пользователь прописан в локальный конфиг (~/.kube/config, блок users)
 * пользователь может просматривать логи подов и их конфигурацию (kubectl logs pod <pod_id>, kubectl describe pod <pod_id>)
```
ubuntu@devkub-vm:~$ kubectl logs hello-world-deployment-d7ff4699c-7zzmx
ubuntu@devkub-vm:~$ kubectl describe pod hello-world-deployment-d7ff4699c-7zzmx
Name:         hello-world-deployment-d7ff4699c-7zzmx
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Mon, 15 Aug 2022 12:42:10 +0000
Labels:       app=hello-world
              pod-template-hash=d7ff4699c
Annotations:  <none>
Status:       Running
IP:           172.17.0.3
IPs:
  IP:           172.17.0.3
Controlled By:  ReplicaSet/hello-world-deployment-d7ff4699c
Containers:
  hello-world:
    Container ID:   docker://78799ca2d33a4c638f297ee788f3d1b8c519f84996628b6bdd0d163722554fb0
    Image:          k8s.gcr.io/echoserver:1.4
    Image ID:       docker-pullable://k8s.gcr.io/echoserver@sha256:5d99aa1120524c801bc8c1a7077e8f5ec122ba16b6dda1a5d3826057f67b9bcb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 15 Aug 2022 12:42:13 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nz9m4 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-nz9m4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  9m21s  default-scheduler  Successfully assigned default/hello-world-deployment-d7ff4699c-7zzmx to minikube
  Normal  Pulling    9m20s  kubelet            Pulling image "k8s.gcr.io/echoserver:1.4"
  Normal  Pulled     9m18s  kubelet            Successfully pulled image "k8s.gcr.io/echoserver:1.4" in 1.884828804s
  Normal  Created    9m18s  kubelet            Created container hello-world
  Normal  Started    9m18s  kubelet            Started container hello-world
```

## Задание 3: Изменение количества реплик 
Поработав с приложением, вы получили запрос на увеличение количества реплик приложения для нагрузки. Необходимо изменить запущенный deployment, увеличив количество реплик до 5. Посмотрите статус запущенных подов после увеличения реплик. 

Требования:
 * в deployment из задания 1 изменено количество реплик на 5
 * проверить что все поды перешли в статус running (kubectl get pods)
```
ubuntu@devkub-vm:~$ kubectl apply -f hello-world-deployment.yaml 
deployment.apps/hello-world-deployment configured
или
ubuntu@devkub-vm:~$ kubectl scale deployment/hello-world-deployment --replicas=5
ubuntu@devkub-vm:~$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
hello-world-deployment-d7ff4699c-54rq5   1/1     Running   0          9s
hello-world-deployment-d7ff4699c-7zzmx   1/1     Running   0          7m26s
hello-world-deployment-d7ff4699c-8ppr9   1/1     Running   0          7m26s
hello-world-deployment-d7ff4699c-gb4ds   1/1     Running   0          9s
hello-world-deployment-d7ff4699c-wkfw9   1/1     Running   0          9s
```
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
