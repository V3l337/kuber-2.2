# Задание 1 - Создать Deployment приложения, использующего локальный PV, созданный вручную

## Создать Deployment приложения, состоящего из контейнеров busybox и multitool. Создать PV и PVC для подключения папки на локальной ноде, которая будет использована в поде.
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-busybox-multitool
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busy-multi
  template:
    metadata:
      labels:
        app: busy-multi
    spec: 
      containers:
      - name: busybox
        image: busybox
        command: ["/bin/sh", "-c", "while true; do echo \"$(date) - Log entry\" >> /mnt/data/log.txt; sleep 5; done"]
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt/data
      - name: multitool
        image: wbitt/network-multitool
        command: ["/bin/sh", "-c", "sleep infinity"]
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt/data
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: example-pvc

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath: 
    path: /mnt/data

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  resources:
    requests:
      storage: 500Mi
  accessModes:
    - ReadWriteOnce
```

## Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории.

![Продемонстрировать, что multitool может читать файл, в который busybox пишет каждые пять секунд в общей директории](https://github.com/user-attachments/assets/05c03f07-6c97-4d42-85df-051a33e5442e)

## Удалить Deployment и PVC. Продемонстрировать, что после этого произошло с PV. Пояснить, почему.
### Потому что persistentVolumeReclaimPolicy: Retain означает, что даже после удаления PVC данные сохраняются.

![Удалить Deployment и PVC  Продемонстрировать, что после этого произошло с PV  Пояснить, почему](https://github.com/user-attachments/assets/25e2cb09-3b0c-4628-ae61-9343430c1e75)

## Продемонстрировать, что файл сохранился на локальном диске ноды. Удалить PV. Продемонстрировать что произошло с файлом после удаления PV. Пояснить, почему.
### Удаление PV не затрагивает файлы на hostPath. Это потому, что Kubernetes управляет только PV, а не фактическими данными на узле.

![Продемонстрировать, что файл сохранился на локальном диске ноды](https://github.com/user-attachments/assets/ef718899-556e-4497-b7be-95d618ed9d80)

![Удалить PV  Продемонстрировать что произошло с файлом после удаления PV  Пояснить, почему](https://github.com/user-attachments/assets/433d6052-5fa8-4dce-b703-61ac573ff532)


# Задание 2 - Создать Deployment приложения, которое может хранить файлы на NFS с динамическим созданием PV.

## Включить и настроить NFS-сервер на MicroK8S.Создать Deployment приложения состоящего из multitool, и подключить к нему PV, созданный автоматически на сервере NFS.
```bash
microk8s enable storage
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        volumeMounts:
        - name: nfs-volume
          mountPath: /tmp
      volumes:
      - name: nfs-volume
        persistentVolumeClaim:
          claimName: mypvc-nfs

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc-nfs
spec:
  resources:
    requests:
      storage: 500Mi
  storageClassName: my-nfs
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany

---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-nfs
provisioner: microk8s.io/hostpath
```

## Продемонстрировать возможность чтения и записи файла изнутри пода.

![Продемонстрировать возможность чтения и записи файла изнутри пода](https://github.com/user-attachments/assets/17adb064-8a73-45e6-8517-d352f2ddbd63)








