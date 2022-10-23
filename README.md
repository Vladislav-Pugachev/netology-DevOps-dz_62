###Манифесты
- [backend](./prod/DeploymentBackend.yml)
- [frontend](./prod/DeploymentFrontend.yml)
- [postgres](./prod/StatefulSet.yml)
- [PV](./prod/PersistentVolume.yml)
- [PVC](./prod/PersistentVolumeClaim.yml)
- [StorageClass](./prod/StorageClass.yml)

### Задание 1: проверить работоспособность каждого компонента
- port-forward
- - Backend
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl port-forward services/backend 9000
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
Handling connection for 9000


vlad@home:~/Documents/Netology/DevOps/dz_№62$ curl localhost:9000
{"detail":"Not Found"}
```
- - Forntend
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl port-forward services/front 8000
Forwarding from 127.0.0.1:8000 -> 80
Forwarding from [::1]:8000 -> 80
Handling connection for 8000


vlad@home:~/Documents/Netology/DevOps/dz_№62$ curl localhost:8000
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Список</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="/build/main.css" rel="stylesheet">
</head>
<body>
    <main class="b-page">
        <h1 class="b-page__title">Список</h1>
        <div class="b-page__content b-items js-list"></div>
    </main>
    <script src="/build/main.js"></script>
</body>
</html>v
```
- - Postgres

```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl port-forward services/postgres 5432
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432
Handling connection for 5432



vlad@home:~/Documents/Netology/DevOps/dz_№62$ psql -U postgres -h localhost
psql (13.8 (Debian 13.8-0+deb11u1))
Введите "help", чтобы получить справку.

postgres=# \l
                                 Список баз данных
    Имя    | Владелец | Кодировка | LC_COLLATE |  LC_CTYPE  |     Права доступа     
-----------+----------+-----------+------------+------------+-----------------------
 db        | postgres | UTF8      | en_US.utf8 | en_US.utf8 | 
 postgres  | postgres | UTF8      | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8      | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |           |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8      | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |           |            |            | postgres=CTc/postgres
(4 строки)

postgres=# 

```

- exec
- - Backend
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl exec -it backend-79df7c55d-p6jsp --  psql -U postgres -h postgres
Password for user postgres: 
psql (11.17 (Debian 11.17-0+deb10u1), server 13.8)
WARNING: psql major version 11, server major version 13.
         Some psql features might not work.
Type "help" for help.

postgres=# 
postgres=# 
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 db        | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(4 rows)

postgres=# 
```

- - Frontend
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl exec -it front-7cbf67d575-p6jlc -- curl backend:9000 
{"detail":"Not Found"}
```

- - Postgres
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl exec -it postgres-0 -- psql -U postgres
psql (13.8)
Type "help" for help.

postgres=# 
```
### Задание 2: ручное масштабирование

- До манипуляции
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl get pods 
NAME                        READY   STATUS    RESTARTS   AGE
backend-79df7c55d-cvp2q     1/1     Running   0          95s
front-7cbf67d575-p6jlc      1/1     Running   0          70m
multitool-c55978665-k55rf   1/1     Running   0          70m
multitool-c55978665-nkrms   1/1     Running   0          70m
postgres-0                  1/1     Running   0          69m
```

- scale 3
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl scale --replicas=3-f prod/DeploymentBackend.yml
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl scale --replicas=3 -f prod/DeploymentFrontnd.yml

vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl get pods 
NAME                        READY   STATUS    RESTARTS   AGE
backend-79df7c55d-5tw5g     1/1     Running   0          15s
backend-79df7c55d-9256d     1/1     Running   0          15s
backend-79df7c55d-cvp2q     1/1     Running   0          3m12s
front-7cbf67d575-2x8vl      1/1     Running   0          7s
front-7cbf67d575-9lwj2      1/1     Running   0          7s
front-7cbf67d575-p6jlc      1/1     Running   0          71m
multitool-c55978665-k55rf   1/1     Running   0          71m
multitool-c55978665-nkrms   1/1     Running   0          71m
postgres-0                  1/1     Running   0          71m

vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP               NODE      NOMINATED NODE   READINESS GATES
backend-79df7c55d-5tw5g     1/1     Running   0          82s     10.233.105.135   worker1   <none>           <none>
backend-79df7c55d-9256d     1/1     Running   0          82s     10.233.125.8     worker2   <none>           <none>
backend-79df7c55d-cvp2q     1/1     Running   0          4m19s   10.233.105.134   worker1   <none>           <none>
front-7cbf67d575-2x8vl      1/1     Running   0          74s     10.233.125.9     worker2   <none>           <none>
front-7cbf67d575-9lwj2      1/1     Running   0          74s     10.233.105.136   worker1   <none>           <none>
front-7cbf67d575-p6jlc      1/1     Running   0          72m     10.233.125.3     worker2   <none>           <none>
multitool-c55978665-k55rf   1/1     Running   0          73m     10.233.125.2     worker2   <none>           <none>
multitool-c55978665-nkrms   1/1     Running   0          73m     10.233.105.129   worker1   <none>           <none>
postgres-0                  1/1     Running   0          72m     10.233.105.130   worker1   <none>           <none>
vlad@home:~/Documents/Netology/DevOps/dz_№62$ 

```

- scale 1
```commandline
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl scale --replicas=1-f prod/DeploymentBackend.yml 
vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl scale --replicas=1 -f prod/DeploymentFrontend.yml 

vlad@home:~/Documents/Netology/DevOps/dz_№62$ kubectl get pods -o wide 
NAME                        READY   STATUS    RESTARTS   AGE     IP               NODE      NOMINATED NODE   READINESS GATES
backend-79df7c55d-9256d     1/1     Running   0          4m37s   10.233.125.8     worker2   <none>           <none>
front-7cbf67d575-9lwj2      1/1     Running   0          4m29s   10.233.105.136   worker1   <none>           <none>
multitool-c55978665-k55rf   1/1     Running   0          76m     10.233.125.2     worker2   <none>           <none>
multitool-c55978665-nkrms   1/1     Running   0          76m     10.233.105.129   worker1   <none>           <none>
postgres-0                  1/1     Running   0          75m     10.233.105.130   worker1   <none>           <none>
```