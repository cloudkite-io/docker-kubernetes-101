# Set up MySQL and Wordpress on Minikube

### Define a secret password for mysql
```bash
kubectl create secret generic mysql-pass --from-literal=password=YOUR_PASSWORD
```


### Set up MySQL
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
#apiVersion: v1
#kind: PersistentVolumeClaim
#metadata:
#  name: mysql-pv-claim
#  labels:
#    app: wordpress
#spec:
#  accessModes:
#    - ReadWriteOnce
#  resources:
#    requests:
#      storage: 20Gi
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress-mysql
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
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
#        volumeMounts:
#        - name: mysql-persistent-storage
#          mountPath: /var/lib/mysql
#      volumes:
#      - name: mysql-persistent-storage
#        persistentVolumeClaim:
#          claimName: mysql-pv-claim
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
#  type: LoadBalancer
  type: NodePort
---
#apiVersion: v1
#kind: PersistentVolumeClaim
#metadata:
#  name: wp-pv-claim
#  labels:
#    app: wordpress
#spec:
#  accessModes:
#    - ReadWriteOnce
#  resources:
#    requests:
#      storage: 20Gi
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: wordpress
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
      - image: wordpress
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
#        volumeMounts:
#        - name: wordpress-persistent-storage
#          mountPath: /var/www/html
#      volumes:
#      - name: wordpress-persistent-storage
#        persistentVolumeClaim:
#          claimName: wp-pv-claim
```

Get service URL
```bash
minikube service wordpress --url
```
