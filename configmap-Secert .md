Kubernetes ConfigMap and Secret Integration with MongoDB and Spring Boot App
📘 Overview
This document explains how to use ConfigMap and Secret in a Kubernetes environment to manage configuration data and sensitive credentials for an application. It covers:

What ConfigMaps and Secrets are

How they work with Pods

A real-world application using:

MongoDB ReplicaSet in the prod namespace

Spring Boot Deployment in the dev namespace

ConfigMaps and Secrets for secure and dynamic configuration

📂 What is a ConfigMap?
A ConfigMap is used to store non-confidential configuration data in key-value pairs. It allows you to decouple configuration artifacts from image content.

Stored in plain text in etcd

Suitable for things like environment variables, command-line arguments, and config files

Not encrypted

yaml
Copy
Edit
apiVersion: v1
kind: ConfigMap
metadata:
  name: springappconfig
  namespace: prod
data:
  db_username: devdb
🔐 What is a Secret?
A Secret is used to store sensitive data, such as passwords, OAuth tokens, and SSH keys.

Encoded in base64

Stored securely in etcd

Kubernetes provides built-in type like Opaque for generic secrets

yaml
Copy
Edit
apiVersion: v1
kind: Secret
metadata:
  name: springappsecret
  namespace: prod
type: Opaque
stringData:
  db_password: devdb@123
🗃 MongoDB ReplicaSet Using ConfigMap & Secret
This ReplicaSet manages the MongoDB Pod and consumes data from both ConfigMap and Secret.

yaml
Copy
Edit
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mongodb
  namespace: prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongocon
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            configMapKeyRef:
              name: springappconfig
              key: db_username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: springappsecret
              key: db_password
        volumeMounts:
        - name: mongonfsvol
          mountPath: /data/db
      volumes:
      - name: mongonfsvol
        persistentVolumeClaim:
          claimName: mongodb1-pvc
🧱 MongoDB Service
yaml
Copy
Edit
apiVersion: v1
kind: Service
metadata:
  name: mongosvc
  namespace: prod
spec:
  type: ClusterIP
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
🗄 PersistentVolume & Claim for MongoDB
yaml
Copy
Edit
# PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 172.31.11.218
    path: /mnt/nfs_share
---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb1-pvc
  namespace: prod
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
☕ Spring Boot App Deployment (consumes MongoDB)
yaml
Copy
Edit
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springapp
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: springapp
  template:
    metadata:
      labels:
        app: springapp
    spec:
      containers:
      - name: springapp
        image: kkeducation12345/spring-app:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: MONGO_DB_HOSTNAME
          value: mongosvc
        - name: MONGO_DB_USERNAME
          value: devdb
        - name: MONGO_DB_PASSWORD
          value: devdb@123
        resources:
          requests:
            cpu: 300m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
🌐 Spring Boot Service
yaml
Copy
Edit
apiVersion: v1
kind: Service
metadata:
  name: springappsvc
  namespace: dev
spec:
  type: NodePort
  selector:
    app: springapp
  ports:
  - port: 80
    targetPort: 8080
🧪 Testing and Verification
View Environment Variables in Pod
bash
Copy
Edit
kubectl exec -it configmap-demo-pod -- env
Describe Pod Volume Info
bash
Copy
Edit
kubectl describe pod configmap-demo-pod
Check Mounted Files
bash
Copy
Edit
kubectl exec -it configmap-demo-pod -- ls /config
kubectl exec -it configmap-demo-pod -- cat /config/game.properties
⚠️ Caution
ConfigMaps are not encrypted → Use only for non-sensitive data.

Secrets are base64 encoded, not encrypted → Use external tools like SealedSecrets or HashiCorp Vault for stronger security.

✅ Summary
Resource Type	Purpose	Stored In	Security
ConfigMap	Non-sensitive configuration	etcd (plain text)	❌ No encryption
Secret	Sensitive data like passwords	etcd (base64)	✔️ More secure
PVC + PV	Persistent storage	NFS	✔️ Retains data
Service	Network access to Pods	N/A	N/A
Deployment/RS	App and DB management	N/A	N/A

Let me know if you want this as a downloadable .md file.









Ask ChatGPT

