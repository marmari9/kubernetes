# Kubernetes Research

## Why is Kubernetes Needed?
Kubernetes is an open-source platform designed to automate the deployment, scaling, and management of containerized applications. Modern applications often use microservices, which run in containers. Managing these containers at scale requires automation, which Kubernetes provides.

## Benefits of Kubernetes
- **Scalability**: Adjusts container numbers based on demand.
- **Portability**: Works across on-premises, hybrid, and multiple clouds.
- **High Availability**: Ensures application stability through redundancy.
- **Resource Efficiency**: Optimizes resource allocation.
- **Declarative Configuration**: Defines desired state and self-heals.

## Success Stories
- **Pokemon Go**: Managed massive user load effectively.
- **Airbnb**: Improved deployment efficiency.
- **New York Times**: Enhanced content delivery.

## Kubernetes Architecture
Kubernetes follows a **master-worker architecture** with a control plane managing worker nodes.

### Control Plane Components
- **kube-apiserver**: Exposes the API.
- **etcd**: Stores cluster data.
- **kube-scheduler**: Assigns pods to nodes.
- **kube-controller-manager**: Manages controllers.

### Worker Node Components
- **kubelet**: Ensures containers run properly.
- **kube-proxy**: Manages networking.
- **Container Runtime**: Runs containers (Docker).

## The Cluster Setup

### What is a Cluster?
A Kubernetes cluster consists of a **control plane** and multiple **worker nodes** to run applications.

### Master vs. Worker Nodes
- **Master Node (Control Plane)**: Manages scheduling and scaling.
- **Worker Nodes**: Run applications and workloads.

### Managed vs. Self-Managed Kubernetes
#### Managed (e.g., GKE, EKS)
**Pros:**
- Simplifies management
- Easier scaling
- Cloud integration

**Cons:**
- Higher cost
- Limited control

#### Self-Managed Kubernetes
**Pros:**
- Full customization
- Cost efficiency

**Cons:**
- Complex setup
- Higher operational overhead

### Control Plane vs. Data Plane
- **Control Plane**: Manages cluster state.
- **Data Plane**: Runs workloads.

## Kubernetes Objects
### Common Kubernetes Objects
- **Pods**: The smallest deployable unit.
- **ReplicaSets**: Maintain a set number of running pods.
- **Deployments**: Manage ReplicaSets and updates.

### What Does It Mean That a Pod is "Ephemeral"?
Pods are **temporary** and can be replaced anytime by Kubernetes if needed.

## Security Concerns with Containers
- **Network Security**: Use **Network Policies**.
- **Least Privilege Access**: Implement **RBAC**.
- **Image Security**: Scan and sign images.
- **Runtime Security**: Use tools like **Falco**.

## Maintained Images
### What Are Maintained Images?
Container images that are regularly updated and supported.

### Pros and Cons
**Pros:**
- Security updates
- Stability
- Community support

**Cons:**
- Limited customization
- May include unwanted dependencies


# Task: Get Kubernetes Running Using Docker Desktop

## **Step 1: Check if Kubernetes is Running**
1. Open **Git Bash** (Windows).
2. Run the following command to check if Kubernetes is active:
   ```sh
   kubectl get service
   ```
3. If Kubernetes is **not running**, you will see an error message indicating that it cannot connect to the cluster.
4. Leave this terminal window open for later steps.

---

## **Step 2: Enable Kubernetes in Docker Desktop**
1. **Open Docker Desktop**.
2. Click on the **Settings icon** in the top-right corner.
3. Navigate to the **Kubernetes** tab.
4. **Enable Kubernetes** by checking the box.
5. Click **Apply & Restart**.

![alt text](<enable kubernetes.png>)

---

## **Step 3: Verify Kubernetes is Running**
1. Once Docker Desktop restarts, go back to the terminal window.
2. Run the following command again:
   ```sh
   kubectl get nodes
   ```
3. If Kubernetes is running, you should see a node named `docker-desktop` in the output.


![alt text](<task 1.png>)

---

## **Step 4: Update `kubectl` Configuration**
1. Ensure that `kubectl` is pointing to the correct Kubernetes context:
   ```sh
   kubectl config current-context
   ```
   - If the output is **not** `docker-desktop`, update it with:
     ```sh
     kubectl config use-context docker-desktop
     ```
![alt text](<task 12.png>)

---

# Kubernetes Tasks Summary
## Task: Create Nginx Deployment Only

## **Step 1: Create a YAML file**
Create a new file named `nginx-deploy.yml`.

## **Step 2: Define the Deployment in YAML**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: daraymonsta/nginx-257:dreamteam
        ports:
        - containerPort: 80
```

## **Step 3: Apply the Deployment**
Run the following command to create the deployment:
```sh
kubectl apply -f nginx-deploy.yml
```

## **Step 4: Get Deployment Details**
Run the following commands to retrieve details:

- **Deployment details:**
  ```sh
  kubectl get deployment nginx-deployment
  ```


  ![alt text](deployment.png)

- **ReplicaSets details:**
  ```sh
  kubectl get rs
  ```
  ![alt text](rs.png)

- **Pods details:**
  ```sh
  kubectl get pods
  ```
  ![alt text](<get pods-1.png>)

  
- **All three at once:**
  ```sh
  kubectl get all
  ```
  ![alt text](<get all.png>)

## **Step 5: Connect to Deployment in Browser**
### **Using ClusterIP (Default)**
1. Run:
   ```sh
   kubectl expose deployment nginx-deployment --type=ClusterIP --port=80
   ```
2. Check the service:
   ```sh
   kubectl get svc nginx-deployment
   ```
   ![alt text](<no external ips.png>)

3. Test inside the cluster:
   ```sh
   kubectl run test-pod --image=busybox -it --rm --restart=Never -- wget -O- nginx-deployment
   ```


   ![](<works inside cluster only.png>)

### **If You Can’t Connect, Possible Reasons:**
- **ClusterIP services are only accessible within the cluster.**, therefore no external IP is available. use port forwarding or expose the deployment using Nodeport Service.
- **Firewall restrictions blocking the port.** 
- **Pods are not running or in a crash loop.**

---
# Task: Create a NodePort Service for Nginx

## **Step 1: Create a YAML file**
Create a new file named `nginx-service.yml` inside your repository `tech501-k8s` under the subfolder:

```
k8s-yaml-definitions/local-nginx-deploy/
```

## **Step 2: Define the NodePort Service in YAML**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30001
```

## **Step 3: Apply the Service**
Run the following command to create the service:
```sh
kubectl apply -f nginx-service.yml
```

## **Step 4: Verify Service is Running**
Run the following command to check the services:
```sh
kubectl get svc nginx-svc
```


  ![alt text](nodeport.png)

## **Step 5: Test Access in Browser**
Open your web browser and go to:
```
http://localhost:30001
```

  ![alt text](<nodeport 30001.png>)
Or use your node’s IP:
```
http://<node-ip>:30001
```

---

### **Task: See What Happens When a Pod is Deleted**

1. **Listed running pods:**
    ```sh
    kubectl get pods
    ```

2. **Deleted one of the pods:**
    ```sh
    kubectl delete pod <pod-name>
    ```    
3. **Checked if a new pod was created:**
    ```sh
    kubectl get pods
    ```


   ![alt text](<pods recreate after deletion.png>)
    
4. **Fetched detailed information about the newest pod:**
    ```sh
    kubectl describe pod <pod-name>
    ```
   ![](<newest pod created.png>)
---

### **Task: Scale Deployment Without Downtime**

#### **Method 1: Edit the Deployment in Real-Time**
```sh
kubectl edit deployment nginx-deployment
# Changed replicas: 4
```

#### **Method 2: Apply an Updated YAML File**
```sh
# Edited nginx-deploy.yml to replicas: 5
kubectl apply -f nginx-deploy.yml
```

#### **Method 3: Scale via Command**
```sh
kubectl scale deployment nginx-deployment --replicas=6
```

---

### **Task: Delete Kubernetes Deployments and Services**

```sh
kubectl delete -f nginx-deploy.yml
kubectl delete -f nginx-service.yml
```

---

### **Task: Deploy NodeJS Sparta Test App in Kubernetes**

#### **Step 1: Created Deployment & Service YAMLs**

**`nodejs-deploy.yml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-sparta-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nodejs-sparta
  template:
    metadata:
      labels:
        app: nodejs-sparta
    spec:
      containers:
      - name: nodejs-sparta
        image: mrmri9/sparta-node:v1
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          value: "mongodb://mongo-service:27017/sparta"
```

**`nodejs-service.yml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodejs-sparta-svc
spec:
  type: NodePort
  selector:
    app: nodejs-sparta
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30002
```

#### **Step 2: Applied the Deployment & Service**
```sh
kubectl apply -f nodejs-deploy.yml
kubectl apply -f nodejs-service.yml
```

#### **Step 3: Verified the App is Running**
```sh
kubectl get pods
kubectl get services
```
Accessed the app via `http://localhost:30002`.

---

### **Task: Add MongoDB Deployment & Persistent Volume**

**`mongo-deploy.yml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:latest
        ports:
        - containerPort: 27017
        volumeMounts:
        - mountPath: /data/db
          name: mongo-storage
      volumes:
      - name: mongo-storage
        persistentVolumeClaim:
          claimName: mongo-pvc
```

#### **Step 4: Created Persistent Volume & Claim**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ""
  volumeName: mongo-pv
```

#### **Step 5: Applied and Verified MongoDB Data Retention**
```sh
kubectl apply -f mongo-deploy.yml
kubectl apply -f mongo-pvc.yml
kubectl exec -it mongo-db-XXXXX -- mongosh
use sparta
db.posts.find().pretty()
```

---

### **Task: Horizontal Pod Autoscaler (HPA) for NodeJS App**

1. **Configured Resource Limits in `nodejs-deploy.yml`**
```yaml
resources:
  requests:
    cpu: "100m"
  limits:
    cpu: "500m"
```
2. **Deployed the HPA**
```sh
kubectl autoscale deployment nodejs-sparta-app --cpu-percent=50 --min=2 --max=10
```
3. **Tested Load Using Apache Bench (ab)**
```sh
docker run --rm jordi/ab -n 5000 -c 50 http://host.docker.internal:30002/posts
```
4. **Checked HPA Scaling**
```sh
kubectl get hpa -w
```













# **Minikube Multi-App Deployment Log**

## **Objective**
Deploy **three applications** on a single cloud instance running **Minikube**, ensuring that each app is accessible externally via different endpoints.

## **Tasks Completed Today**

### **1️ Setting Up Minikube on EC2**
- Installed Minikube with `none` driver (since we are running it directly on EC2).
- Installed required dependencies:
  - `kubectl`
  - `conntrack`
  - `crictl`
  - `docker`
- Configured Minikube to start with:
  ```bash
  minikube start --driver=none
  ```

### **2️ Deploying First Application (NodePort Service)**
- **App Image:** `daraymonsta/nginx-257:dreamteam`
- **Replicas:** 5
- **Exposed via NodePort (`30001`)**
- Created **Deployment YAML (`app1-deployment.yaml`)**:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-app
  spec:
    replicas: 5
    selector:
      matchLabels:
        app: nginx-app
    template:
      metadata:
        labels:
          app: nginx-app
      spec:
        containers:
        - name: nginx-container
          image: daraymonsta/nginx-257:dreamteam
          ports:
          - containerPort: 80
  ```
- Created **Service YAML (`app1-service.yaml`)**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    type: NodePort
    selector:
      app: nginx-app
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
        nodePort: 30001
  ```
- Deployed the app:
  ```bash
  kubectl apply -f app1-deployment.yaml
  kubectl apply -f app1-service.yaml
  ```


### **3️ Configuring Nginx Reverse Proxy for First App**
- Installed Nginx and configured the reverse proxy to **expose the first app externally on `http://<EC2-public-IP>`**.
- Updated `/etc/nginx/sites-available/default`:
  ```nginx
  server {
      listen 80;
      server_name _;
      location / {
          proxy_pass http://192.168.49.2:30001;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }
  ```
- Restarted Nginx:
  ```bash
  sudo systemctl restart nginx
  ```
- **Verified External Access:**
  ```bash
  curl http://<EC2-public-IP>
  ```




### **4️ Deploying Second Application (LoadBalancer Service)**
- **App Image:** `daraymonsta/tech201-nginx-auto:v1`
- **Replicas:** 2
- **Exposed via LoadBalancer (`port 9000`, NodePort `30002`)**
- Created **Deployment YAML (`app2-deployment.yaml`)**:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: app2-nginx
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: app2-nginx
    template:
      metadata:
        labels:
          app: app2-nginx
      spec:
        containers:
        - name: app2-container
          image: daraymonsta/tech201-nginx-auto:v1
          ports:
          - containerPort: 80
  ```
- Created **Service YAML (`app2-service.yaml`)**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: app2-service
  spec:
    type: LoadBalancer
    selector:
      app: app2-nginx
    ports:
      - protocol: TCP
        port: 9000
        targetPort: 80
        nodePort: 30002
  ```
- Deployed the app:
  ```bash
  kubectl apply -f app2-deployment.yaml
  kubectl apply -f app2-service.yaml
  ```
- **Started Minikube Tunnel to Support LoadBalancer Services**:
  ```bash
  minikube tunnel
  ```
- **Verified Internal Access:**
  ```bash
  curl http://10.106.186.126:9000
  ```


![alt text](<task 3 part 2.png>)



### **5️ Configuring Nginx Reverse Proxy for Second App**
- Updated `/etc/nginx/sites-available/default`:
  ```nginx
  server {
      listen 9000;
      server_name _;
      location / {
          proxy_pass http://10.106.186.126:9000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }
  ```
- Restarted Nginx:
  ```bash
  sudo systemctl restart nginx
  ```
- **Verified External Access:**
  ```bash
  curl http://<EC2-public-IP>:9000
  ```

---

## **Challenges & Blockers Faced Today**

### **1️ Minikube Start Failed Due to Missing Dependencies**
**Issue:**
- Minikube required `conntrack`, `crictl`, and `docker` but they were missing.
**Solution:**
- Installed missing packages:
  ```bash
  sudo apt install -y conntrack cri-tools docker.io
  ```

### **2️ External Access to First App (NodePort) Failing**
**Issue:**
- The Nginx reverse proxy was misconfigured, causing `502 Bad Gateway` errors.
**Solution:**
- Fixed the Nginx config and restarted the service.

### **3️ LoadBalancer Service Wasn’t Assigning an External IP**
**Issue:**
- The `EXTERNAL-IP` for the LoadBalancer service remained in `<pending>` state.
**Solution:**
- Enabled Minikube tunnel:
  ```bash
  minikube tunnel
  ```

### **4️ Nginx Not Forwarding Requests to Second App**
**Issue:**
- Reverse proxy was set to `localhost:9000`, but the LoadBalancer service was using `10.106.186.126`.
**Solution:**
- Changed `proxy_pass` to:
  ```nginx
  proxy_pass http://10.106.186.126:9000;
  ```
- Restarted Nginx and tested external access.

---


# Deploying a Containerized 2-Tier Application on Minikube

## Overview
This guide provides step-by-step instructions to deploy a 2-tier application (Node.js app and MongoDB database) on a single VM using Minikube. The deployment includes persistent storage, autoscaling, and external access via a reverse proxy.

## Prerequisites
- An EC2 instance with Ubuntu 22.04
- Docker installed
- Minikube installed and configured to use Docker as the driver
- kubectl installed
- Nginx installed for reverse proxy

## Step 1: Start Minikube
Ensure Minikube starts automatically on VM reboot by creating a systemd service.

### Create Systemd Service for Minikube

```ini
sudo nano /etc/systemd/system/minikube.service
```

Add the following content:

```ini
[Unit]
Description=Minikube
After=network.target docker.service
Requires=docker.service

[Service]
User=ubuntu
ExecStartPre=/bin/sleep 15  # Ensures Docker is ready before starting Minikube
ExecStart=/usr/local/bin/minikube start --driver=docker --memory=1900 --cpus=2 --wait=all
ExecStop=/usr/local/bin/minikube stop
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```sh
sudo systemctl daemon-reload
sudo systemctl enable minikube
sudo systemctl restart minikube
```

Reboot the instance and verify Minikube starts automatically:

```sh
sudo reboot
```

After reboot:

```sh
minikube status
kubectl get nodes
```

![alt text](<minikube work after reboot.png>)



## Step 2: Configure Persistent Volume for MongoDB
Create a Persistent Volume (PV) and Persistent Volume Claim (PVC) to provide 100MB storage for MongoDB.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

Apply the configuration:

```sh
kubectl apply -f mongo-pv.yaml
```

## Step 3: Deploy MongoDB
Create a MongoDB deployment using the persistent storage.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:5.0
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-storage
          mountPath: /data/db
      volumes:
      - name: mongo-storage
        persistentVolumeClaim:
          claimName: mongo-pvc
```

Apply the configuration:

```sh
kubectl apply -f mongo-deployment.yaml
```

## Step 4: Deploy the Node.js Application
Create a deployment for the application and configure Horizontal Pod Autoscaler (HPA).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sparta-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sparta
  template:
    metadata:
      labels:
        app: sparta
    spec:
      containers:
      - name: sparta-app
        image: mrmri9/sparta-node:v2
        ports:
        - containerPort: 3000
        resources:
          requests:
            cpu: "50m"
            memory: "64Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
        env:
        - name: DB_HOST
          value: "mongodb://mongo-service:27017/mydatabase"
```

Apply the deployment:

```sh
kubectl apply -f sparta-app.yaml
```

## Step 5: Configure HPA
Create an HPA to scale the application from 2 to 10 replicas based on CPU usage.

```sh
kubectl autoscale deployment sparta-app --cpu-percent=50 --min=2 --max=10
```

Verify HPA is working:

```sh
kubectl get hpa
kubectl describe hpa sparta-app
```

Run a load test to validate scaling:

```sh
kubectl run -it --rm load-gen --image=busybox -- /bin/sh -c "while true; do wget -q -O- http://sparta-service.default.svc.cluster.local; done"
```

Check if replicas increase:

```sh
kubectl get pods
kubectl top pods
```

![alt text](<HPA scalling .png>)

![alt text](<scaling dropped.png>)


## Step 6: Expose the Application via NodePort
Create a service to expose the app.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sparta-service
spec:
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30008
  selector:
    app: sparta
```

Apply the service:

```sh
kubectl apply -f sparta-service.yaml
```

Find the Minikube IP:

```sh
minikube ip
```

Access the app at:

```sh
http://<MINIKUBE_IP>:30008
```

## Step 7: Set Up Nginx Reverse Proxy
Edit the Nginx config:

```sh
sudo nano /etc/nginx/sites-available/default
```

Add:

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://<MINIKUBE_IP>:30008;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Restart Nginx:

```sh
sudo systemctl restart nginx
```

Now, access the app via the public IP of the EC2 instance:

```sh
http://<EC2_PUBLIC_IP>
```


