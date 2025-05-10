# Домашнее задание к занятию «Базовые объекты K8S» - Мельник Юрий Александрович

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Pod с приложением и подключиться к нему со своего локального компьютера. 

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным Git-репозиторием.

------
### Подготовка к выполению задания 

1. проверим установлен ли 
 ```
 docker --version
 ```
 получен ответ **Docker version 28.1.1, build 4eba377**
 установка доккер не требуеться!  

2. Установка kubectl
 ```
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
 chmod +x kubectl
 sudo mv kubectl /usr/local/bin/
 kubectl version --client
 ```

3. Установка Minikube (для локального кластера)
```
 curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
 sudo install minikube-linux-amd64 /usr/local/bin/minikube
 minikube version
```
 запуск minikube
```
minikube start --driver=docker
```
![img 1](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img1.jpg)
![img 2](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img2.jpg)

Создание пространства для работы
```
mkdir -p ~/k8s-homework
cd ~/k8s-homework
```

4. проверка готовности 
```
minikube status
kubectl get nodes
docker ps
```

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. Описание [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) и примеры манифестов.
2. Описание [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

------

### Задание 1. Создать Pod с именем hello-world

1. Создать манифест (yaml-конфигурацию) Pod.
2. Использовать image - gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Подключиться локально к Pod с помощью `kubectl port-forward` и вывести значение (curl или в браузере).

------

### Решение 1.
перйдем в рабочую директорию
```
cd ~/k8s-homework
nano hello-world-pod.yaml
```
1. Создадим Манифест 
Содеримое файла hello-world-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: echoserver
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
```
2. Подключение к Pod через port-forward
```
kubectl port-forward pod/hello-world 8080:8080
```

в другом терминале выполним 
```
curl http://localhost:8080
```

![img 3](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img3.jpg)
![img 4](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img4.jpg)

3. Освободим порт,  убив процесс
```
sudo lsof -i -P -n | grep 8080
[sudo] пароль для lamer: 
firefox   2336849           lamer   93u  IPv4 16997878      0t0  TCP 127.0.0.1:45016->127.0.0.1:8080 (ESTABLISHED)
kubectl   2343691           lamer    7u  IPv4 16988109      0t0  TCP 127.0.0.1:8080 (LISTEN)
kubectl   2343691           lamer    8u  IPv4 16998874      0t0  TCP 127.0.0.1:8080->127.0.0.1:45016 (ESTABLISHED)
curl      2347261           lamer    5u  IPv4 16997252      0t0  TCP 127.0.0.1:33532->127.0.0.1:8080 (ESTABLISHED)
lamer@lamer-VirtualBox:~/k8s-homework$ sudo kill -9 2343691
lamer@lamer-VirtualBox:~/k8s-homework$ sudo lsof -i -P -n | grep 8080
[1]+  Убито              kubectl port-forward pod/hello-world 8080:8080
lamer@lamer-VirtualBox:~/k8s-homework$ sudo lsof -i -P -n | grep 8080
lamer@lamer-VirtualBox:~/k8s-homework$ 
```

![img 5](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img5.jpg)

### Задание 2. Создать Service и подключить его к Pod

1. Создать Pod с именем netology-web.
2. Использовать image — gcr.io/kubernetes-e2e-test-images/echoserver:2.2.
3. Создать Service с именем netology-svc и подключить к netology-web.
4. Подключиться локально к Service с помощью `kubectl port-forward` и вывести значение (curl или в браузере).


### Решение 2. Создать Service и подключить его к Pod
1. перйдем в рабочую директорию
```
cd ~/k8s-homework
nano netology-web.yaml
```

листинг netology-web.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: netology-web
  labels:
    app: netology-web
spec:
  containers:
  - name: echoserver
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: netology-svc
spec:
  selector:
    app: netology-web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

2. применим манифест
```
kubectl apply -f netology-web.yaml
```  

3. Проверим, что Pod и Service создались
```
kubectl get pods
kubectl get svc
```

4.  Подключение через port-forward
```
kubectl port-forward svc/netology-svc 8080:80
```

5.  Проверка подключения
в другом терминале терминале выполни:
curl http://localhost:8080

![img 6](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img6.jpg)
![img 7](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img7.jpg)
![img 8](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img8.jpg)

### Правила приёма работы

6.  Удаление всех ресурсов из netology-web.yaml
```
kubectl delete -f netology-web.yaml
kubectl get pods
kubectl get svc
```
![img 9](https://github.com/ysatii/kuber-homeworks1.2/blob/main/img/img9.jpg)


1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода команд `kubectl get pods`, а также скриншот результата подключения.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------

### Критерии оценки
Зачёт — выполнены все задания, ответы даны в развернутой форме, приложены соответствующие скриншоты и файлы проекта, в выполненных заданиях нет противоречий и нарушения логики.

На доработку — задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки.
