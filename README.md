To achieve a highly available MariaDB cluster on Kubernetes with data synchronization and automatic reconnection for clients, i am showing an example of the Galera Cluster. allowing all nodes to be both read and write nodes. Below are the YAML files and a brief description of the approach

Approach:
Create Persistent Volumes (PVs) and Persistent Volume Claims (PVCs):

# pv.yaml
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mariadb-pv-0
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: your-storage-class
  hostPath:
    path: "/path/to/host/folder-0"
```
# pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pvc-0
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: your-storage-class
  resources:
    requests:
      storage: 1Gi
```
Deploy the MariaDB StatefulSet:

Create a StatefulSet YAML file for MariaDB. This file ensures that each MariaDB instance gets its own identity and stable network identity.
# mariadb-statefulset.yaml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb
spec:
  serviceName: mariadb
  replicas: 3
  selector:
    matchLabels:
      app: mariadb
  template:
    metadata:
      labels:
        app: mariadb
    spec:
      containers:
      - name: mariadb
        image: mariadb:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: your-root-password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mariadb-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: your-storage-class
      resources:
        requests:
          storage: 1Gi
```
Deploy a Service for MariaDB:

Create a Service YAML file to expose the MariaDB instances internally.


# mariadb-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mariadb
spec:
  clusterIP: None
  ports:
  - port: 3306
    name: mysql
  selector:
    app: mariadb
```
# mariadb-service-external.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mariadb-external
spec:
  type: LoadBalancer
```
# mariadb-galera-statefulset.yaml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mariadb-galera
spec:
  serviceName: mariadb-galera
  replicas: 3
  selector:
    matchLabels:
      app: mariadb-galera
  template:
    metadata:
      labels:
        app: mariadb-galera
    spec:
      containers:
      - name: mariadb
        image: mariadb:galera
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: your-root-password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mariadb-persistent-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: your-storage-class
      resources:
        requests:
          storage: 1Gi
```
# mariadb-galera-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mariadb-galera
spec:
  clusterIP: None
  ports:
  - port: 3306
    name: mysql
  selector:
    app: mariadb-galera
```
# mariadb-galera-service-external.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mariadb-galera-external
spec:
  type: LoadBalancer
  ports:
  - port: 3306
    name: mysql
  selector:
    app: mariadb-galera

  ports:
  - port: 3306
    name: mysql
  selector:
    app: mariadb
```

