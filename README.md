# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1» - Балдин
 
### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.

Манифест Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-hw4
  namespace: hw4
spec:
  selector:
    matchLabels:
      app: nginx-multitool
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env: 
          - name: HTTP_PORT
            value: "8080"
```

![1.1](jpeg/1.jpg)

2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.

Манифест Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: int-service
  namespace: hw4
spec:
  selector:
    app: nginx-multitool
  ports:
    - protocol: TCP
      name: nginx
      port: 9001
      targetPort: 80
    - protocol: TCP
      name: multitool
      port: 9002
      targetPort: 8080
```

![1.2](jpeg/2.jpg)

3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

![1.3](jpeg/3.jpg) 

![1.4](jpeg/4.jpg) 

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

Манифест:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nodeport
  namespace: hw4
spec:
  type: NodePort
  selector:
    app: nginx-multitool
  ports:
    - port: 80
      name: nginx
      targetPort: 80
      nodePort: 30001
    - port: 8080
      name: multitool
      targetPort: 8080
      nodePort: 30002
```

![2.1](jpeg/5.jpg) 
------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
