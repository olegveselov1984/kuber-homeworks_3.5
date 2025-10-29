# Домашнее задание к занятию Troubleshooting

### Цель задания

Устранить неисправности при деплое приложения.

### Чеклист готовности к домашнему заданию

1. Кластер K8s.

### Задание. При деплое приложение web-consumer не может подключиться к auth-db. Необходимо это исправить

1. Установить приложение по команде:
```shell
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```
2. Выявить проблему и описать.
3. Исправить проблему, описать, что сделано.
4. Продемонстрировать, что проблема решена.

Решение:  
Перенес код к себе и запустил:
```
ubuntu@ubuntu:~/src/kuber/3.5/kuber-homeworks_3.5$ kubectl apply -f task.yaml 
Error from server (NotFound): error when creating "task.yaml": namespaces "web" not found
Error from server (NotFound): error when creating "task.yaml": namespaces "data" not found
Error from server (NotFound): error when creating "task.yaml": namespaces "data" not found
```

Добавляем в файл создание Namespace
```
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: web
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: data
---
```
Запускаем файл и смотрим результат:
```
ubuntu@ubuntu:~/src/kuber/3.5/kuber-homeworks_3.5$ kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
data          auth-db-fb7c59b7f-p6vdv         1/1     Running   0          65s
kube-system   calico-node-88gl7               1/1     Running   0          29m
web           web-consumer-5f5b45b976-bdvlt   1/1     Running   0          65s
web           web-consumer-5f5b45b976-nrk7m   1/1     Running   0          65s
ubuntu@ubuntu:~/src/kuber/3.5/kuber-homeworks_3.5$ 
```

Pods создались, посмотрим логи:

```
ubuntu@ubuntu:~/src/kuber/3.5/kuber-homeworks_3.5$ kubectl logs deployments/web-consumer -n web
Found 2 pods, using pod/web-consumer-5f5b45b976-bdvlt
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
curl: (6) Couldn't resolve host 'auth-db'
```
Проверяю сервисы kubectl get service -A
```
ubuntu@ubuntu:~/src/kuber/3.5/kuber-homeworks_3.5$ kubectl get service -A
NAMESPACE   NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
data        auth-db      ClusterIP   10.152.183.54   <none>        80/TCP    18m
default     kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP   95m
```
У сервиса auth-db IP 10.152.183.54

Заходим в контейнер и проверяем доступность 
kubectl exec -n web web-consumer-5f5b45b976-bdvlt -it -- /bin/sh
```
[ root@web-consumer-5f5b45b976-bdvlt:/ ]$ curl auth-db
curl: (6) Couldn't resolve host 'auth-db'
[ root@web-consumer-5f5b45b976-bdvlt:/ ]$ 
[ root@web-consumer-5f5b45b976-bdvlt:/ ]$ curl 10.152.183.54
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[ root@web-consumer-5f5b45b976-bdvlt:/ ]$ 
```

отсутствует доступ по имени, но есть доступ по ip
Вместо auth-db указал ip адрес 10.152.183.54 и все заработало.
```
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

PS
пытался указать имя, не работает:
auth-db.data
auth-db.data.svc.cluster.local
```
ubuntu@ubuntu:~/src/kuber/3.5/kuber-homeworks_3.5$ kubectl get pods  -n web
NAME                            READY   STATUS    RESTARTS   AGE
web-consumer-55646866b7-jgmsp   1/1     Running   0          17m
web-consumer-55646866b7-rg6dz   1/1     Running   0          17m
ubuntu@ubuntu:~/src/kuber/3.5/kuber-homeworks_3.5$ kubectl exec -n web web-consumer-55646866b7-rg6dz -it -- /bin/sh
/bin/sh: shopt: not found
[ root@web-consumer-55646866b7-rg6dz:/ ]$ 
[ root@web-consumer-55646866b7-rg6dz:/ ]$ 
[ root@web-consumer-55646866b7-rg6dz:/ ]$ 
[ root@web-consumer-55646866b7-rg6dz:/ ]$ 
[ root@web-consumer-55646866b7-rg6dz:/ ]$ 
[ root@web-consumer-55646866b7-rg6dz:/ ]$ 
[ root@web-consumer-55646866b7-rg6dz:/ ]$ nslookup 10.152.183.54
Server:    10.152.183.10
Address 1: 10.152.183.10

Name:      10.152.183.54
Address 1: 10.152.183.54
[ root@web-consumer-55646866b7-rg6dz:/ ]$ 
```
Проблема явно в DNS но не нашел как ее решить. Только если всех загнать в один namespase или отредактировать hosts.
Прощу подсказать более красивое решение.

<img width="954" height="487" alt="image" src="https://github.com/user-attachments/assets/c251599c-f4df-4b22-b68b-960c76ec5f67" />



### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
