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
  name: nginx-svc
spec:
  ports:
    - name: web
      port: 80
  selector:
    app: dep1
```
![image](https://github.com/askarpoff/kuber_ex3/assets/108946489/afe8f515-aa2a-4784-86ee-5ce669775347)

6. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

