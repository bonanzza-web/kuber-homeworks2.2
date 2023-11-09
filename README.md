# Домашнее задание к занятию «Хранение в K8s. Часть 2»

### Цель задания

В тестовой среде Kubernetes нужно создать PV и продемострировать запись и хранение файлов.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

### Дополнительные материалы для выполнения задания

1. [Инструкция по установке NFS в MicroK8S](https://microk8s.io/docs/nfs). 
2. [Описание Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). 
3. [Описание динамического провижининга](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/). 
4. [Описание Multitool](https://github.com/wbitt/Network-MultiTool).

------

### Задание 1

**Что нужно сделать**

Создать Deployment приложения, использующего локальный PV, созданный вручную.

1. Создать Deployment приложения, состоящего из контейнеров busybox и multitool.
2. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
3. Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории. 
4. Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
5. Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV.  Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------

### Ответы:

```
ubuntu@server-kube:~$ micro get deploy
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
busy-mult   1/1     1            1           2m11s
ubuntu@server-kube:~$ micro get pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvcshka   Bound    pvshka   1Gi        RWO            manual         2m12s
ubuntu@server-kube:~$ micro get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
pvshka   1Gi        RWO            Retain           Bound    default/pvcshka   manual                  2m19s
ubuntu@server-kube:~$ micro get po
NAME                         READY   STATUS    RESTARTS   AGE
busy-mult-5ddcf9cf8c-fzwvx   2/2     Running   0          5m
ubuntu@server-kube:~$ micro exec -it busy-mult-5ddcf9cf8c-fzwvx -c multitools -- bash
busy-mult-5ddcf9cf8c-fzwvx:/# tail -15 /tmp/logs/logs.txt 
It is a log -- Thu Nov  9 09:47:13 UTC 2023
It is a log -- Thu Nov  9 09:47:18 UTC 2023
It is a log -- Thu Nov  9 09:47:23 UTC 2023
It is a log -- Thu Nov  9 09:47:28 UTC 2023
It is a log -- Thu Nov  9 09:47:33 UTC 2023
It is a log -- Thu Nov  9 09:47:38 UTC 2023
It is a log -- Thu Nov  9 09:47:43 UTC 2023
It is a log -- Thu Nov  9 09:47:48 UTC 2023
It is a log -- Thu Nov  9 09:47:53 UTC 2023
It is a log -- Thu Nov  9 09:47:58 UTC 2023
It is a log -- Thu Nov  9 09:48:03 UTC 2023
It is a log -- Thu Nov  9 09:48:08 UTC 2023
It is a log -- Thu Nov  9 09:48:13 UTC 2023
It is a log -- Thu Nov  9 09:48:18 UTC 2023
It is a log -- Thu Nov  9 09:48:23 UTC 2023
busy-mult-5ddcf9cf8c-fzwvx:/# exit
exit
ubuntu@server-kube:~$ ls /data/pvolume/
logs.txt
ubuntu@server-kube:~$ micro delete -f deploy.yaml 
deployment.apps "busy-mult" deleted
ubuntu@server-kube:~$ micro delete -f pvc.yaml 
persistentvolumeclaim "pvcshka" deleted
ubuntu@server-kube:~$ micro get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM             STORAGECLASS   REASON   AGE
pvshka   1Gi        RWO            Retain           Released   default/pvcshka   manual                  7m56s
ubuntu@server-kube:~$ tail -15 /data/pvolume/logs.txt 
It is a log -- Thu Nov  9 09:49:13 UTC 2023
It is a log -- Thu Nov  9 09:49:18 UTC 2023
It is a log -- Thu Nov  9 09:49:23 UTC 2023
It is a log -- Thu Nov  9 09:49:28 UTC 2023
It is a log -- Thu Nov  9 09:49:33 UTC 2023
It is a log -- Thu Nov  9 09:49:38 UTC 2023
It is a log -- Thu Nov  9 09:49:43 UTC 2023
It is a log -- Thu Nov  9 09:49:48 UTC 2023
It is a log -- Thu Nov  9 09:49:53 UTC 2023
It is a log -- Thu Nov  9 09:49:58 UTC 2023
It is a log -- Thu Nov  9 09:50:03 UTC 2023
It is a log -- Thu Nov  9 09:50:08 UTC 2023
It is a log -- Thu Nov  9 09:50:13 UTC 2023
It is a log -- Thu Nov  9 09:50:19 UTC 2023
It is a log -- Thu Nov  9 09:50:24 UTC 2023    

```
PV остался после удаления деплоймента и pvc, т.к. он не зависит от подов и pvc, в этом и есть его основная функция выжить когда все упадет, в моем понимании.    
```
ubuntu@server-kube:~$ micro delete -f pv.yaml 
persistentvolume "pvshka" deleted
ubuntu@server-kube:~$ micro get pv
No resources found
ubuntu@server-kube:~$ ls /data/pvolume 
logs.txt
ubuntu@server-kube:~$ tail -15 /data/pvolume/logs.txt 
It is a log -- Thu Nov  9 09:49:13 UTC 2023
It is a log -- Thu Nov  9 09:49:18 UTC 2023
It is a log -- Thu Nov  9 09:49:23 UTC 2023
It is a log -- Thu Nov  9 09:49:28 UTC 2023
It is a log -- Thu Nov  9 09:49:33 UTC 2023
It is a log -- Thu Nov  9 09:49:38 UTC 2023
It is a log -- Thu Nov  9 09:49:43 UTC 2023
It is a log -- Thu Nov  9 09:49:48 UTC 2023
It is a log -- Thu Nov  9 09:49:53 UTC 2023
It is a log -- Thu Nov  9 09:49:58 UTC 2023
It is a log -- Thu Nov  9 09:50:03 UTC 2023
It is a log -- Thu Nov  9 09:50:08 UTC 2023
It is a log -- Thu Nov  9 09:50:13 UTC 2023
It is a log -- Thu Nov  9 09:50:19 UTC 2023
It is a log -- Thu Nov  9 09:50:24 UTC 2023
```
Файл остался после удаления pv, т.к. в нем использовался persistentVolumeReclaimPolicy: Retain, что позволяет сохранить файлы после удаления pv. Если бы было Delete файл удалился бы.
------

### Задание 2

**Что нужно сделать**

Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

1. Включить и настроить NFS-сервер на MicroK8S.
2. Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
3. Продемонстрировать возможность чтения и записи файла изнутри пода. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------


### Ответы:

```
ubuntu@server-kube:~$ micro get pvc
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-nfs   Bound    pvc-c391da8f-3a49-4e61-a39b-1e2c077e85a9   1Gi        RWO            sc-nfs         3m58s
ubuntu@server-kube:~$ micro get po -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
nfs-deploy-5fc684b46f-fbddn   1/1     Running   0          26s   10.1.55.200   server-kube   <none>           <none>
ubuntu@server-kube:~$ micro exec -it nfs-deploy-5fc684b46f-fbddn -- bash
nfs-deploy-5fc684b46f-fbddn:/# ls
bin         dev         etc         lib         mnt         proc        root        sbin        sys         usr
certs       docker      home        media       opt         pvc-volume  run         srv         tmp         var
nfs-deploy-5fc684b46f-fbddn:/# echo "Але смольный, у вас пиво есть?" > /pvc-volume/smolniy.txt
nfs-deploy-5fc684b46f-fbddn:/# cat /pvc-volume/smolniy.txt 
Але смольный, у вас пиво есть?
nfs-deploy-5fc684b46f-fbddn:/# exit
exit
ubuntu@server-kube:~$ sudo ls /srv/nfs/pvc-c391da8f-3a49-4e61-a39b-1e2c077e85a9
smolniy.txt
ubuntu@server-kube:~$ sudo cat /srv/nfs/pvc-c391da8f-3a49-4e61-a39b-1e2c077e85a9/smolniy.txt 
Але смольный, у вас пиво есть?
ubuntu@server-kube:~$ micro delete pod nfs-deploy-5fc684b46f-fbddn
pod "nfs-deploy-5fc684b46f-fbddn" deleted
ubuntu@server-kube:~$ micro get po
NAME                          READY   STATUS    RESTARTS   AGE
nfs-deploy-5fc684b46f-dkb6z   1/1     Running   0          9s
ubuntu@server-kube:~$ micro exec -it nfs-deploy-5fc684b46f-dkb6z -- cat /pvc-volume/smolniy.txt
Але смольный, у вас пиво есть?
ubuntu@server-kube:~$ micro get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                  STORAGECLASS   REASON   AGE
data-nfs-server-provisioner-0              1Gi        RWO            Retain           Bound    nfs-server-provisioner/data-nfs-server-provisioner-0                           20h
pvc-c391da8f-3a49-4e61-a39b-1e2c077e85a9   1Gi        RWO            Delete           Bound    default/pvc-nfs                                        sc-nfs                  11m
```

------

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.