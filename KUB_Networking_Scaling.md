# Kubernetes Networking and Scaling - Complete Beginner's Guide

**A comprehensive guide explaining every concept with practical examples on AWS EKS**

---

## Table of Contents
1. [Services (ClusterIP, NodePort, LoadBalancer)](#part-1-kubernetes-services)
2. [Ingress Controllers](#part-2-ingress-controllers)
3. [Storage (PV, PVC, StorageClass)](#part-3-kubernetes-storage)
4. [Horizontal Pod Autoscaling](#part-4-horizontal-pod-autoscaling)
5. [Vertical Pod Autoscaling](#part-5-vertical-pod-autoscaling)
6. [Load Balancing in EKS](#part-6-load-balancing-in-eks)
7. [Helm Package Manager](#part-7-helm-package-manager)
8. [Production Best Practices](#part-8-production-best-practices)

---

## Prerequisites Verification

```bash
kubectl cluster-info                      # Check connection
kubectl get nodes                         # Should show 2 nodes
kubectl get deployment nginx-deployment   # Should show 5/5 replicas
kubectl get pods -l app=nginx            # Should show 5 running pods
```

---

## Part 1: Kubernetes Services

### What is a Service?

**The Problem:**
- Pods have temporary IP addresses
- When a pod restarts, it gets a new IP
- How do other applications reliably connect to your pods?

**The Solution:**
- Services provide a stable IP and DNS name
- Traffic is automatically load-balanced across pods
- Services remain even when pods change

**Analogy:** Services are like a company's main phone number—it stays the same even when employees (pods) change.

### Service Types

| Type | Access From | Use Case | Cost |
|------|------------|----------|------|
| ClusterIP | Inside cluster only | Internal microservices | Free |
| NodePort | Internet via Node IP:Port | Development/testing | Free |
| LoadBalancer | Internet via Load Balancer DNS | Production | ~$18/mo |

---

### 1.1 ClusterIP - Internal Communication

**When to use:** Database, cache, internal APIs that shouldn't be exposed externally.

**Create ClusterIP Service:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx          # Routes to pods with this label
  ports:
  - port: 8080          # Service port
    targetPort: 80      # Pod port
EOF
```

**Verify:**
```bash
kubectl get svc nginx-clusterip
kubectl describe svc nginx-clusterip
kubectl get endpoints nginx-clusterip   # Shows pod IPs
```

**Test from inside cluster:**
```bash
# Create temporary test pod
kubectl run curl-test --image=curlimages/curl -i --tty --rm -- sh

# Inside the pod:
curl http://nginx-clusterip:8080
# Works! Kubernetes DNS resolves the service name

exit  # Pod auto-deletes
```

---

### 1.2 NodePort - Development Access

**When to use:** Development, testing, small projects without load balancer.

**Create NodePort:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080     # Port on each node (30000-32767)
EOF
```

**Access format:** `http://<NODE-IP>:30080`

**Get node IP:**
```bash
kubectl get nodes -o wide   # Look at EXTERNAL-IP column
```

**⚠️ EKS requires security group configuration:**
```bash
# Allow port 30080
SECURITY_GROUP=$(aws ec2 describe-security-groups \
    --filters "Name=tag:aws:eks:cluster-name,Values=my-eks-cluster" \
    --query 'SecurityGroups[0].GroupId' --output text --region us-east-1)

aws ec2 authorize-security-group-ingress \
    --group-id $SECURITY_GROUP \
    --protocol tcp --port 30080 --cidr 0.0.0.0/0 --region us-east-1
```

---

### 1.3 LoadBalancer - Production Access

**When to use:** Production applications that need public access.

**Create LoadBalancer:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
EOF
```

**Wait for AWS to provision (2-3 minutes):**
```bash
kubectl get svc nginx-loadbalancer -w
# Press Ctrl+C when EXTERNAL-IP appears
```

**Get URL and test:**
```bash
LB_URL=$(kubectl get svc nginx-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "URL: http://$LB_URL"
curl http://$LB_URL
```

---

## Part 2: Ingress Controllers

### What is Ingress?

**The Problem:**
- Each LoadBalancer service costs $18/month
- 10 services = $180/month just for load balancers
- Managing multiple load balancers is complex

**The Solution:**
- Ingress: One load balancer for many services
- Routes traffic based on URL path or hostname
- Costs only $18/month for unlimited services

**Example:**
```
Without Ingress:
Internet → LB ($18) → API Service
Internet → LB ($18) → Web Service  
Internet → LB ($18) → Admin Service
Total: $54/month

With Ingress:
Internet → One LB ($18) → Ingress → API Service
                                  → Web Service
                                  → Admin Service
Total: $18/month
```

### Ingress Features

1. **Path-based routing:** `/api` → API service, `/web` → Web service
2. **Host-based routing:** `api.example.com` → API, `web.example.com` → Web
3. **SSL/TLS termination:** HTTPS handled at load balancer
4. **URL rewrites and redirects**
5. **Authentication and authorization**

---

### 2.1 Install AWS Load Balancer Controller

**What it does:** Automatically creates and manages AWS Application Load Balancers for your Ingress resources.

**Step 1: Create OIDC provider**
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 --cluster my-eks-cluster --approve
```

**Step 2: Create IAM policy**
```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.0/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

**Step 3: Create service account**
```bash
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

eksctl create iamserviceaccount \
    --cluster=my-eks-cluster \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --region us-east-1 --approve
```

**Step 4: Install with Helm**
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=my-eks-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller
```

**Verify:**
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

---

### 2.2 Basic Ingress

**Create ClusterIP service (required):**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
EOF
```

**Create Ingress:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
EOF
```

**Get URL:**
```bash
kubectl get ingress nginx-ingress -w  # Wait for ADDRESS
INGRESS_URL=$(kubectl get ingress nginx-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "URL: http://$INGRESS_URL"
```

---

### 2.3 Path-Based Routing

**Use case:** Route different URL paths to different services.

**Create second service:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: httpd
        image: httpd:2.4
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: apache-service
spec:
  type: ClusterIP
  selector:
    app: apache
  ports:
  - port: 80
EOF
```

**Create multi-path Ingress:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-path-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
  - http:
      paths:
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
      - path: /apache
        pathType: Prefix
        backend:
          service:
            name: apache-service
            port:
              number: 80
EOF
```

**Test:**
```bash
MULTI_URL=$(kubectl get ingress multi-path-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$MULTI_URL/nginx
curl http://$MULTI_URL/apache
```

---

## Part 3: Kubernetes Storage

### What is Storage in Kubernetes?

**The Problem:**
- Containers are ephemeral (temporary)
- When a pod dies, all data inside it is lost
- Databases, user uploads, logs need to persist

**The Solution:**
- Kubernetes provides persistent storage that survives pod restarts
- Storage is independent of pod lifecycle

### Storage Components

1. **StorageClass (SC):** Defines types of storage available
2. **PersistentVolume (PV):** Actual storage (like an AWS EBS disk)
3. **PersistentVolumeClaim (PVC):** Request for storage

**Workflow:**
```
You create PVC → StorageClass provisions PV → Pod uses PVC
```

---

### 3.1 Create StorageClass

**What it defines:**
- Storage type (SSD, HDD)
- Performance characteristics
- Provider (AWS EBS, NFS, etc.)

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
EOF
```

**Verify:**
```bash
kubectl get sc
kubectl describe sc ebs-gp3
```

---

### 3.2 Create and Use PVC

**Create PVC:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
  - ReadWriteOnce      # One pod at a time
  storageClassName: ebs-gp3
  resources:
    requests:
      storage: 5Gi
EOF
```

**Use PVC in deployment:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-with-storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-storage
  template:
    metadata:
      labels:
        app: nginx-storage
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
        - name: storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: nginx-pvc
EOF
```

**Test persistence:**
```bash
POD=$(kubectl get pod -l app=nginx-storage -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD -- bash -c "echo 'Persistent!' > /usr/share/nginx/html/index.html"
kubectl delete pod $POD
sleep 30
NEW_POD=$(kubectl get pod -l app=nginx-storage -o jsonpath='{.items[0].metadata.name}')
kubectl exec $NEW_POD -- cat /usr/share/nginx/html/index.html
# Data persisted!
```

---

### 3.3 StatefulSet

**What is StatefulSet?**
- For stateful applications (databases, queues)
- Provides stable pod names (web-0, web-1)
- Each pod gets its own persistent storage

**When to use:**
- Databases (PostgreSQL, MySQL, MongoDB)
- Message queues (Kafka, RabbitMQ)
- Any app needing stable identity

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-stateful
spec:
  clusterIP: None
  selector:
    app: nginx-stateful
  ports:
  - port: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: nginx-stateful
  replicas: 3
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ebs-gp3
      resources:
        requests:
          storage: 1Gi
EOF
```

**Verify:**
```bash
kubectl get statefulset web
kubectl get pods -l app=nginx-stateful    # Note: web-0, web-1, web-2
kubectl get pvc                           # Each pod has its own PVC
```

---

## Part 4: Horizontal Pod Autoscaling

### What is HPA?

**Definition:** Automatically scales the number of pods based on metrics.

**How it works:**
```
CPU usage > 50% → Add more pods
CPU usage < 50% → Remove pods
```

**Use case:** Handle traffic spikes automatically.

---

### 4.1 Install Metrics Server

**What it does:** Collects CPU/memory metrics from pods.

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify (wait 1-2 minutes)
kubectl get deployment metrics-server -n kube-system
kubectl top nodes
kubectl top pods
```

---

### 4.2 Set Resource Requests

**Why needed:** HPA needs to know resource requests to calculate percentages.

```bash
kubectl patch deployment nginx-deployment -p '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "nginx",
          "resources": {
            "requests": {"cpu": "100m", "memory": "128Mi"},
            "limits": {"cpu": "200m", "memory": "256Mi"}
          }
        }]
      }
    }
  }
}'
```

---

### 4.3 Create HPA

**Simple HPA:**
```bash
kubectl autoscale deployment nginx-deployment \
    --cpu-percent=50 --min=2 --max=10
```

**Advanced HPA:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
EOF
```

**Monitor:**
```bash
kubectl get hpa -w
kubectl describe hpa nginx-hpa
```

---

### 4.4 Test HPA

**Generate load:**
```bash
# Terminal 1: Watch HPA
kubectl get hpa nginx-hpa -w

# Terminal 2: Generate load
kubectl run load-generator --image=busybox --rm -it -- \
  /bin/sh -c "while true; do wget -q -O- http://nginx-service; done"

# Watch pods scale up
# Stop load (Ctrl+C) and watch scale down
```

---

## Part 5: Vertical Pod Autoscaling

### What is VPA?

**Definition:** Automatically adjusts CPU/memory requests/limits for pods.

**Difference from HPA:**
- HPA: Changes number of pods
- VPA: Changes resources of existing pods

---

### 5.1 Install VPA

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
cd ../../..

# Verify
kubectl get pods -n kube-system | grep vpa
```

---

### 5.2 Create VPA

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vpa-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: vpa-demo
  template:
    metadata:
      labels:
        app: vpa-demo
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        resources:
          requests:
            cpu: "50m"
            memory: "64Mi"
---
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-demo
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vpa-demo
  updatePolicy:
    updateMode: "Off"    # Options: Off, Initial, Auto
EOF
```

**Check recommendations (wait 2-3 min):**
```bash
kubectl get vpa
kubectl describe vpa vpa-demo
```

---

## Part 6: Load Balancing in EKS

### Types of Load Balancing

1. **Service Load Balancing:** Automatic across pods
2. **Ingress Load Balancing:** ALB for multiple services
3. **Node Load Balancing:** ELB/NLB distributes to nodes

### Health Checks

```yaml
# Add to deployment
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 15
  periodSeconds: 20
```

---

## Part 7: Helm Package Manager

### What is Helm?

**Definition:** Package manager for Kubernetes (like apt/yum for Linux).

**Why use Helm:**
1. **Templating:** Reuse YAML with different values
2. **Versioning:** Easy rollback to previous versions
3. **Dependency management:** Install apps with dependencies
4. **Sharing:** Use pre-built charts from community

**Analogy:** 
- Manual YAML = Building furniture from scratch
- Helm = Buying IKEA furniture with instructions

---

### 7.1 Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

---

### 7.2 Using Helm

**Add repository:**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

**Search charts:**
```bash
helm search repo nginx
helm search hub wordpress
```

**Install a chart:**
```bash
helm install my-nginx bitnami/nginx
# Install with custom values
helm install my-nginx bitnami/nginx --set replicaCount=3
```

**List installed:**
```bash
helm list
```

**Upgrade:**
```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```

**Rollback:**
```bash
helm rollback my-nginx 1
```

**Uninstall:**
```bash
helm uninstall my-nginx
```

---

### 7.3 Create Your Own Chart

```bash
# Create chart structure
helm create my-app

# Structure:
# my-app/
#   Chart.yaml       # Metadata
#   values.yaml      # Default values
#   templates/       # Kubernetes YAML templates

# Customize values.yaml
cat > my-app/values.yaml <<EOF
replicaCount: 3
image:
  repository: nginx
  tag: "1.21"
service:
  type: LoadBalancer
  port: 80
EOF

# Install
helm install my-release ./my-app

# Package
helm package my-app
```

---

## Part 8: Production Best Practices

### 8.1 Resource Management

**Always set requests and limits:**
```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

**Use ResourceQuotas:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
EOF
```

---

### 8.2 High Availability

**Multiple replicas:**
```yaml
replicas: 3  # Minimum for HA
```

**Pod Anti-Affinity:**
```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        topologyKey: kubernetes.io/hostname
        labelSelector:
          matchLabels:
            app: nginx
```

---

### 8.3 Security

**Non-root user:**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
```

**Network Policies:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx-only
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: allowed
    ports:
    - protocol: TCP
      port: 80
EOF
```

---

### 8.4 Monitoring

**Deploy Prometheus:**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

---

## Cleanup

```bash
# Delete all resources
kubectl delete hpa nginx-hpa
kubectl delete deployment nginx-deployment apache-deployment nginx-with-storage vpa-demo
kubectl delete statefulset web
kubectl delete service nginx-clusterip nginx-nodeport nginx-loadbalancer nginx-service apache-service nginx-stateful
kubectl delete ingress nginx-ingress multi-path-ingress
kubectl delete pvc nginx-pvc www-web-0 www-web-1 www-web-2

# Delete cluster
eksctl delete cluster --name my-eks-cluster --region us-east-1
```

---

## Quick Reference

### Essential Commands
```bash
# Pods
kubectl get pods
kubectl describe pod <n>
kubectl logs <n>
kubectl exec -it <n> -- bash

# Deployments
kubectl get deployments
kubectl scale deployment <n> --replicas=5
kubectl rollout restart deployment <n>

# Services
kubectl get svc
kubectl describe svc <n>

# Storage
kubectl get pvc
kubectl get pv
kubectl get sc

# HPA
kubectl get hpa
kubectl describe hpa <n>

# Ingress
kubectl get ingress
kubectl describe ingress <n>
```

---

**End of Guide** | Practice each section in order for best understanding!
