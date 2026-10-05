# mysql-statefulset-configmap

1) Create configmap

------------------
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config-map
  namespace: dev
data:
  MYSQL_DATABASE: "devops"
  MYSQL_ROOT_PASSWORD: "123456"


kubectl apply -f mysql-configmap.yml
------------------

2) create statefulSet

---------------------
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
  namespace: dev
spec:
  serviceName: "mysql-headless-svc"
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          env:
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: mysql-config-map
                  key: MYSQL_DATABASE
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: mysql-config-map
                  key: MYSQL_ROOT_PASSWORD
          ports:
            - containerPort: 3306
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-storage
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi


kubectl apply -f mysql-statefulset.yml
--------------------

3) Create service 

----------------------
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless-svc
  namespace: dev
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - name: mysql
      port: 3306
      protocol: TCP

kubectl apply -f mysql-headless-svc.yml
---------------------

4) kubectl get all

NAME                      READY   STATUS    RESTARTS   AGE
pod/mysql-statefulset-0   1/1     Running   0          3m5s
pod/mysql-statefulset-1   1/1     Running   0          3m
pod/mysql-statefulset-2   1/1     Running   0          116s

NAME                         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/mysql-headless-svc   ClusterIP   None         <none>        3306/TCP   2m29s

NAME                                 READY   AGE
statefulset.apps/mysql-statefulset   3/3     3m5s


5) kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      5d6h
mysql-config-map   2      13m

6) login into the pod

azhardanish@Mac mysql-statefulset-configmap % k exec -it mysql-statefulset-0 -- mysql -u root -p  
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.46 MySQL Community Server - GPL

Copyright (c) 2000, 2026, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| devops             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.02 sec)

mysql> use devops;
Database changed
mysql> CREATE TABLE Persons (
    ->   PersonID int PRIMARY KEY,
    ->   LastName varchar(255) NOT NULL,
    ->   FirstName varchar(255),
    ->   Address varchar(255),
    ->   City varchar(255)
    -> );
Query OK, 0 rows affected (0.05 sec)

mysql> show tables
    -> ;
+------------------+
| Tables_in_devops |
+------------------+
| Persons          |
+------------------+
1 row in set (0.03 sec)

mysql> exit
Bye


