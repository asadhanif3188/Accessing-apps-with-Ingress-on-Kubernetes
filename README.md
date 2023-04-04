# Accessing Apps with Ingress on Kubernetes

In this project we are going to deploy multiple applications on Kubernetes cluster and Ingress will be used to access those apps from outside the cluster. 

Following Apps would be deployed to demonstrate the working:
1. [Online Shop with Microservices](#1-online-shop-with-microservices) 
2. [Reddit Clone](#2-reddit-clone-app)
3. WordPress with MySQL
4. Odoo with Postgres 

## 1. Online Shop with Microservices
We'll be using the configurations of microservices from the repo [Microservices-based-E-Commerce-App](https://github.com/asadhanif3188/Microservices-based-E-Commerce-App). 

## 2. Reddit Clone App
Following is the configuration used to deploy Reddit Clone App. 

```
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: ClusterIP
  ports:
    - port: 8081
      targetPort: 3000
  selector:
    app: reddit-clone
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  selector:
    matchLabels:
      app: reddit-clone
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - image: asadhanif3188/redditclone:v1
        name: reddit-clone
        ports:
        - containerPort: 3000
```

## 3. WordPress with MySQL
Following is the configuration used to deploy WordPress with MySQL. 

```
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  labels:
    app: wordpress
spec:
  type: ClusterIP
  ports:
    - port: 3306
      targetPort: 3306
  selector:
    app: wordpress
    tier: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
  labels:
    app: wordpress
spec:
  type: ClusterIP
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-deployment
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-service
        - name: WORDPRESS_DB_PASSWORD
          value: password
        ports:
        - containerPort: 80
          name: wordpress
```

## 4. Odoo with Postgres
Following is the configuration used to deploy Odoo with Postgres. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:10.1
          ports:
            - containerPort: 5432
          env:
          - name: POSTGRES_DB
            value: postgresdb
          - name: POSTGRES_USER
            value: admin
          - name: POSTGRES_PASSWORD
            value: root
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  labels:
    app: postgres
spec:
  type: ClusterIP
  ports:
    - port: 5432
  selector:
    app: postgres
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: odoo-deployment
  labels:
    app: odoo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: odoo
  template:
    metadata:
      labels:
        app: odoo
    spec:
      containers:
      - name: odoo
        image: odoo:14.0
        ports:
        - containerPort: 8069
        env:
        - name: HOST
          value: postgres-service
        - name: USER
          value: admin
        - name: PASSWORD
          value: root
---
apiVersion: v1
kind: Service
metadata:    
  name: odoo-service
spec:
  type: ClusterIP
  selector:
    app: odoo
  ports:
    - protocol: TCP
      port: 8069
      targetPort: 8069
```

## Ingress Recource
Following is the configuration for Ingress to access different applications.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapps-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapps.com
    http:
      paths:
      - path: /online-shop
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 8080
      - path: /reddit-clone
        pathType: Prefix
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 8081
      - path: /wordpress
        pathType: Prefix
        backend:
          service:
            name: wordpress-service
            port:
              number: 80
      - path: /odoo
        pathType: Prefix
        backend:
          service:
            name: odoo-service
            port:
              number: 8069
```

## Running Applications and Ingress
It is time to see all the components in action. 

First of all enable the ingress on Minikube, using following command: 

`minikube addons enable ingress`

Run following command to see the status of ingress addon. 

`minikube addons list`

Run following command to activate microservices of online shop app. 

`kubectl apply -f online-shop-microservices.yaml`

Run following command to execute reddit-clone app. 

`kubectl apply -f reddit-clone.yaml`

Run following command to execute WordPress app. 

`kubectl apply -f wordpress-with-mysql.yaml`

Run following command to execute Odoo app. 

`kubectl apply -f odoo-with-postgres.yaml`

Run following command to execute the Ingress resource. 

`kubectl apply -f ingress.yaml`

To see the the IP address of Ingress, run following command. 

`kubectl get ingress`
