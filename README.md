# 05-kuber

# Задача 1-2

Созданы deployment и services для данной задачи front/back
![1](https://github.com/user-attachments/assets/4f4b53e4-14d2-44bf-93b7-4d3b661c2647)

Весь манифест:
```
# Deployment для frontend (nginx)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80

---
# Deployment для backend (multitool)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: multitool
        image: praqma/network-multitool
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "8080"

---
# Service для frontend
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  labels:
    app: frontend
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

---
# Service для backend
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  labels:
    app: backend
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: ClusterIP
```

Взаимодействие чежду ними:

![2](https://github.com/user-attachments/assets/d605c392-91d3-4163-930b-64cc271d8444)

![3](https://github.com/user-attachments/assets/74648b81-fb61-4b4a-b798-7d9624e35432)

# Задача 2

Установден nginx-ingress контроллер который использует metallb для получения ip-address'a сети:

```
 kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

![4](https://github.com/user-attachments/assets/a614001c-6632-4e3b-9262-d397ffb2d686)

Написан манифест который перенаправляет запросы в зависимости от запроса / или /api
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: ""
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```
![5](https://github.com/user-attachments/assets/bdf227c9-94b5-4da9-9d0c-9e3d2f62b374)


Выполнено запросы curl для того что бы убедиться что сервисы доступны один запрос непосредственно с узла ingress второй с узла kuber что бы убедиться что ingress доступен по сети:

![6](https://github.com/user-attachments/assets/aead7a23-95ab-43c5-a180-f254c4f46749)

![7](https://github.com/user-attachments/assets/28dfa024-1a5a-48e7-93b7-810c219b2a01)

