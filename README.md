# Домашнее задание к занятию «Запуск приложений в K8S»

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

------
## Ответ:

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.

Указал другие порты у network-multitool
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dep1
  name: dep1
  namespace: ex3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dep1
  template:
    metadata:
      labels:
        app: dep1
    spec:
      containers:
        - image: nginx:latest
          ports:
            - containerPort: 80
              name: httpnginx
              protocol: TCP
          name: nginx
          imagePullPolicy: IfNotPresent
        - image: wbitt/network-multitool
          name: multitool
          env:
            - name: HTTP_PORT
              value: '8080'
            - name: HTTPS_PORT
              value: '8443'
          imagePullPolicy: IfNotPresent
```
![image](https://github.com/askarpoff/kuber_ex3/assets/108946489/f1b2264b-9831-47c2-8b16-a2db99d4a2ed)

2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.

![image](https://github.com/askarpoff/kuber_ex3/assets/108946489/2ffb9224-e1e2-47c4-bb1f-31b216fe417f)
 
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: dep1-svc
spec:
  ports:
    - name: nginx-http
      port: 80
    - name: multitool-http
      port: 8080
    - name: multitool-https
      port: 8443
  selector:
    app: dep1
```
Проверка доступности через сервис:
```bash
root@learning-k8s:~/kuber_ex3# kubectl port-forward service/dep1-svc 80:8080 --address='0.0.0.0' &
[1] 107088
root@learning-k8s:~/kuber_ex3# Forwarding from 0.0.0.0:80 -> 8080

root@learning-k8s:~/kuber_ex3# curl http://10.1.1.32
Handling connection for 80
WBITT Network MultiTool (with NGINX) - dep1-65b9fb4555-8t28r - 10.1.161.68 - HTTP: 8080 , HTTPS: 8443 . (Formerly praqma/network-multitool)
root@learning-k8s:~/kuber_ex3# killall kubectl
root@learning-k8s:~/kuber_ex3# kubectl port-forward service/dep1-svc 80:80 --address='0.0.0.0' &
[2] 108004
[1]   Terminated              kubectl port-forward service/dep1-svc 80:8080 --address='0.0.0.0'
root@learning-k8s:~/kuber_ex3# Forwarding from 0.0.0.0:80 -> 80

root@learning-k8s:~/kuber_ex3# curl http://10.1.1.32
Handling connection for 80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
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
root@learning-k8s:~/kuber_ex3# killall kubectl
root@learning-k8s:~/kuber_ex3# kubectl port-forward service/dep1-svc 443:8443 --address='0.0.0.0' &
[3] 108440
[2]   Terminated              kubectl port-forward service/dep1-svc 80:80 --address='0.0.0.0'
root@learning-k8s:~/kuber_ex3# Forwarding from 0.0.0.0:443 -> 8443
kubectl port-forward service/decurl http://10.1.1.32^C
root@learning-k8s:~/kuber_ex3# curl https://10.1.1.32 -k
Handling connection for 443
WBITT Network MultiTool (with NGINX) - dep1-65b9fb4555-8t28r - 10.1.161.68 - HTTP: 8080 , HTTPS: 8443 . (Formerly praqma/network-multitool)
root@learning-k8s:~/kuber_ex3# killall kubectl
```

6. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multitool
  labels:
    app: multitool
spec:
  containers:
  - name: multitool
    image: wbitt/network-multitool
    env:
      - name: HTTP_PORT
        value: "1080"
      - name: HTTPS_PORT
        value: "10443"
```
```bash
root@learning-k8s:~/kuber_ex3# kubectl get deployments,svc,pods -o wide
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES                                 SELECTOR
deployment.apps/dep1   2/2     2            2           58m   nginx,multitool   nginx:latest,wbitt/network-multitool   app=dep1

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                    AGE   SELECTOR
service/dep1-svc   ClusterIP   10.152.183.218   <none>        80/TCP,8080/TCP,8443/TCP   22m   app=dep1

NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
pod/multitool               1/1     Running   0          9m15s   10.1.161.65   learning-k8s   <none>           <none>
pod/dep1-65b9fb4555-8t28r   2/2     Running   0          7m54s   10.1.161.68   learning-k8s   <none>           <none>
pod/dep1-65b9fb4555-s5glp   2/2     Running   0          7m53s   10.1.161.71   learning-k8s   <none>           <none>
```
------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

