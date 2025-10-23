# kuber-homeworks_1.3

# Домашнее задание к занятию «Запуск приложений в K8S»

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.
------
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool-deployment
  labels:
    app: nginx-multitool
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
```
<img width="1591" height="341" alt="z1-1" src="https://github.com/user-attachments/assets/a2e9ea10-8973-4122-9afc-815d66d9cc37" />

После запуска видим ошибку, что порт уже прослушивается, так как nginx и multitool по-умолчанию используют порт 80
Требуется перенастроить multitool
```
        env:
        - name: HTTP_PORT
          value: "8080"
```
<img width="810" height="126" alt="z1-2" src="https://github.com/user-attachments/assets/7b909231-d403-4672-bc1c-4f380de56557" />

Устанавливаем replicas: 2 увеличив количество реплик работающего приложения

<img width="817" height="142" alt="z1-3" src="https://github.com/user-attachments/assets/eed19248-9af1-41ad-a20d-ddf98a9c6a59" />

**Service**
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc
spec:
  selector:
    app: nginx-multitool
  ports:
  - name: nginx
    port: 80
    targetPort: 80
  - name: multitool
    port: 8080
    targetPort: 8080
```
<img width="908" height="123" alt="z1-4" src="https://github.com/user-attachments/assets/bcd93066-b522-40b8-896c-17f224926ffe" />

Проверим работу
```
microk8s kubectl exec -it deploy/nginx-multitool-deployment -c multitool -- curl http://nginx-multitool-svc:8080
microk8s kubectl exec -it deploy/nginx-multitool-deployment -c multitool -- curl http://nginx-multitool-svc:80
```

<img width="1635" height="540" alt="z1-5" src="https://github.com/user-attachments/assets/49ed678c-8be5-4e26-bcd8-ef4635b1988b" />

**Pod**
```
apiVersion: v1
kind: Pod
metadata:
  name: multitool-two
spec:
  containers:
  - name: multitool-two
    image: wbitt/network-multitool
```
<img width="1603" height="806" alt="z1-6" src="https://github.com/user-attachments/assets/9f0b403b-8c32-4c70-b1b9-af4ae567c35f" />

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.
------




------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
