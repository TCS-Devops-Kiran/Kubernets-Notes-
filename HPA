📌 What is HPA in Kubernetes?
HPA (Horizontal Pod Autoscaler) is a Kubernetes controller that automatically adjusts the number of pod replicas in a Deployment, ReplicaSet, or StatefulSet based on observed CPU/memory utilization or other custom metrics.

✅ When Do We Use HPA?
Use HPA when your application's workload is variable or unpredictable, and you want to scale automatically based on resource consumption.
⚙️ How It Works
HPA continuously monitors metrics (default is CPU usage via Metrics Server) and:

Increases pod replicas when usage is high

Decreases pod replicas when usage is low

🧪 Example Use Case
Let’s say you have a web application that should maintain CPU usage at 50%. You want to scale between 2 and 10 pods based on that metric.

1. vertical
     2. Horizontal

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50


kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml



metric server set-up
====================

--> search metric server yaml for hpa

--> open the GitHub link

--> kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

--> kubectl get all -n kube-system

--> But pod is not coming up due to certificate issue.

--> kubectl edit deploy metrics-server -n kube-system

--> 

spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls  ---> add this line


=========================================================================

#deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpadeployment
  labels:
    name: hpadeployment
spec:
  replicas: 1
  selector:
    matchLabels:
      name: hpapod
  template:
    metadata:
      labels:
        name: hpapod
    spec:
      containers:
      - name: hpacontainer
        image: k8s.gcr.io/hpa-example
        ports:
        - name: http
          containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "100m"
            memory: "128Mi"
---
#HPA

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpadeploymentautoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpadeployment
  minReplicas: 2
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 30
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

---

#service

apiVersion: v1
kind: Service
metadata:
  name: hpaclusterservice
  labels:
    name: hpaservice
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    name: hpapod
  type: ClusterIP


========================================================


# ==== Execute below commands to increase load====

Now, we will see how the auto scaler reacts to increased load. We will start a container, and send an infinite loop of queries to the php-apache service .
# Create A Temp Pod in interactive mod to access app using service name

  $ kubectl run -i --tty load-generator --rm --image=busybox /bin/sh	
# Execute below command in Temp Pod
 
  $ while true; do wget -q -O- http://hpaclusterservice; done	

Open kubectl terminal in another tab and watch kubectl get pods or kubect get hpa to see how the auto scaler reacts to increased load.

  $ watch kubectl get hpa

3. Apply HPA
bash
Copy
Edit
kubectl autoscale deployment web-app --cpu-percent=50 --min=2 --max=10
This command creates an HPA that:

Keeps CPU usage around 50%

Scales between 2 to 10 pods

4. Check HPA Status
bash
Copy
Edit
kubectl get hpa

