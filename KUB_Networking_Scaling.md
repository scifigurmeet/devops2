# Kubernetes Networking and Scaling - Complete Study Guide

A practical guide covering all Kubernetes networking, storage, scaling, and production concepts with minimal examples.

## Prerequisites

Verify your existing setup:
```bash
kubectl get nodes                    # Should show 2 nodes
kubectl get deployment nginx-deployment  # Should show 5/5 replicas
kubectl get pods -l app=nginx        # Should show 5 running pods
```

---

## Part 1: Kubernetes Services

Services provide stable networking to access your pods.

### 1.1 ClusterIP Service (Internal Only)

**Create ClusterIP service:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80
EOF
```

**Verify:**
```bash
kubectl get service nginx-clusterip
kubectl describe service nginx-clusterip
kubectl get endpoints nginx-clusterip  # Shows pod IPs
```

**Test from inside cluster:**
```bash
# Create temporary pod with curl
kubectl run curl-test --image=curlimages/curl -i --tty --rm -- sh

# Inside the pod shell, run:
curl http://nginx-clusterip:8080

# Exit the pod
exit
```

**Key Points:**
- Only accessible within cluster
- Default service type
- Used for internal microservices communication

---

### 1.2 NodePort Service (External via Node IP)

**Create NodePort service:**
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
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
EOF
```

**Get access details:**
```bash
kubectl get service nginx-nodeport
kubectl get nodes -o wide  # Get node external IPs
```

**Access format:** `http://<NODE-EXTERNAL-IP>:30080`

**Note:** For EKS, you must configure security groups to allow port 30080.

---

### 1.3 LoadBalancer Service (AWS ELB)

**Create LoadBalancer service:**
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
  - protocol: TCP
    port: 80
    targetPort: 80
EOF
```

**Get LoadBalancer URL:**
```bash
# Wait for EXTERNAL-IP (takes 2-3 minutes)
kubectl get service nginx-loadbalancer -w

# Get URL
kubectl get service nginx-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'

# Test
curl http://$(kubectl get service nginx-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

**Service Comparison:**
| Type | Access | Use Case | Cost |
|------|--------|----------|------|
| ClusterIP | Internal only | Microservices | Free |
| NodePort | Node IP + Port | Development | Free |
| LoadBalancer | Public DNS | Production | ~$18/month |

---

## Part 2: Ingress Controllers

Ingress provides HTTP/HTTPS routing using a single load balancer for multiple services.

### 2.1 Install AWS Load Balancer Controller

**Step 1: Create OIDC provider**
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster my-eks-cluster \
    --approve
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
# Get AWS account ID
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create service account
eksctl create iamserviceaccount \
    --cluster=my-eks-cluster \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::$AWS_ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --region us-east-1 \
    --approve
```

**Step 4: Install controller with Helm**
```bash
# Add Helm repo
helm repo add eks https://aws.github.io/eks-charts
helm repo update

# Install controller
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

### 2.2 Basic Ingress Example

**Create ClusterIP service:**
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
    targetPort: 80
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

**Get Ingress URL:**
```bash
# Wait for ADDRESS
kubectl get ingress nginx-ingress -w

# Get URL
kubectl get ingress nginx-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

### 2.3 Path-Based Routing

**Create second application:**
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
    targetPort: 80
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

**Test paths:**
```bash
INGRESS_URL=$(kubectl get ingress multi-path-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$INGRESS_URL/nginx
curl http://$INGRESS_URL/apache
```

---

## Part 3: Kubernetes Storage

### 3.1 Create StorageClass

**Create gp3 StorageClass:**
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
kubectl get storageclass
kubectl describe sc ebs-gp3
```

### 3.2 PersistentVolumeClaim Example

**Create PVC:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
  - ReadWriteOnce
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
# Get pod name
POD=$(kubectl get pod -l app=nginx-storage -o jsonpath='{.items[0].metadata.name}')

# Write data
kubectl exec $POD -- bash -c "echo 'Persistent Data!' > /usr/share/nginx/html/index.html"

# Delete pod
kubectl delete pod $POD

# Wait for new pod
sleep 30
NEW_POD=$(kubectl get pod -l app=nginx-storage -o jsonpath='{.items[0].metadata.name}')

# Verify data persisted
kubectl exec $NEW_POD -- cat /usr/share/nginx/html/index.html
```

### 3.3 StatefulSet Example

**Create StatefulSet:**
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
kubectl get pods -l app=nginx-stateful
kubectl get pvc  # Each pod has its own PVC
```

---

## Part 4: Resource Requests and Limits

**Update nginx-deployment with resources:**
```bash
kubectl patch deployment nginx-deployment -p '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "nginx",
          "resources": {
            "requests": {
              "cpu": "100m",
              "memory": "128Mi"
            },
            "limits": {
              "cpu": "200m",
              "memory": "256Mi"
            }
          }
        }]
      }
    }
  }
}'
```

**Understanding:**
- `requests`: Guaranteed minimum resources
- `limits`: Maximum allowed resources
- `100m` = 0.1 CPU core (100 millicores)
- `128Mi` = 128 mebibytes

**Check resource usage:**
```bash
kubectl top nodes
kubectl top pods -l app=nginx
```

---

## Part 5: Horizontal Pod Autoscaling

### 5.1 Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify
kubectl get deployment metrics-server -n kube-system

# Wait 1-2 minutes, then test
kubectl top nodes
```

### 5.2 Create HPA

**Simple HPA:**
```bash
kubectl autoscale deployment nginx-deployment \
    --cpu-percent=50 \
    --min=2 \
    --max=10
```

**Advanced HPA with YAML:**
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

**Monitor HPA:**
```bash
kubectl get hpa
kubectl get hpa nginx-hpa -w
kubectl describe hpa nginx-hpa
```

### 5.3 Test Autoscaling

**Generate load:**
```bash
# Create load generator
kubectl run load-generator --image=busybox --rm -it -- /bin/sh -c "while true; do wget -q -O- http://nginx-service; done"

# In another terminal, watch scaling
kubectl get hpa nginx-hpa -w
kubectl get pods -l app=nginx -w

# Stop load generator (Ctrl+C)
```

---

## Part 6: Vertical Pod Autoscaling

### 6.1 Install VPA

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
cd ../../..

# Verify
kubectl get pods -n kube-system | grep vpa
```

### 6.2 Create VPA

**Create test deployment:**
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
EOF
```

**Create VPA (recommendation mode):**
```bash
cat <<EOF | kubectl apply -f -
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
    updateMode: "Off"  # Only recommend
EOF
```

**Check recommendations (wait 2-3 minutes):**
```bash
kubectl get vpa
kubectl describe vpa vpa-demo
```

**VPA Update Modes:**
- `Off`: Recommendations only
- `Initial`: Set on pod creation only
- `Auto`: Automatically update running pods

---

## Part 7: Load Balancing in EKS

### 7.1 Service Load Balancing

**LoadBalancer service distributes traffic across pods automatically.**

**Test load distribution:**
```bash
LB_URL=$(kubectl get service nginx-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

# Multiple requests hit different pods
for i in {1..20}; do
  curl -s http://$LB_URL | grep -o "nginx" || echo "Request $i"
done
```

### 7.2 Ingress Load Balancing

**ALB (Application Load Balancer) provides:**
- Path-based routing
- Host-based routing
- SSL termination
- Health checks

**Check ALB health:**
```bash
kubectl describe ingress nginx-ingress
kubectl get ingress nginx-ingress
```

### 7.3 Load Balancing Best Practices

1. **Use readiness probes:**
```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```

2. **Use liveness probes:**
```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 15
  periodSeconds: 20
```

3. **Enable pod disruption budgets:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: nginx
```

---

## Part 8: Helm Package Manager

### 8.1 Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
```

### 8.2 Helm Basics

**Add a repository:**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

**Search for charts:**
```bash
helm search repo nginx
helm search hub wordpress
```

**Install a chart:**
```bash
helm install my-nginx bitnami/nginx

# With custom values
helm install my-nginx bitnami/nginx \
  --set service.type=LoadBalancer \
  --set replicaCount=3
```

**List installed releases:**
```bash
helm list
```

**Upgrade a release:**
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

### 8.3 Create Your Own Chart

**Create chart structure:**
```bash
helm create my-app

# This creates:
# my-app/
#   Chart.yaml          # Chart metadata
#   values.yaml         # Default values
#   templates/          # Kubernetes manifests
#   charts/             # Dependencies
```

**Customize values.yaml:**
```yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.21"
service:
  type: LoadBalancer
  port: 80
```

**Install your chart:**
```bash
helm install my-release ./my-app
```

**Package chart:**
```bash
helm package my-app
# Creates: my-app-0.1.0.tgz
```

---

## Part 9: Production Best Practices

### 9.1 Resource Management

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

**Use ResourceQuotas for namespaces:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: default
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
EOF
```

**Use LimitRanges:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range
spec:
  limits:
  - default:
      cpu: "200m"
      memory: "256Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
    type: Container
EOF
```

### 9.2 High Availability

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

**Multiple availability zones:**
```bash
eksctl create cluster \
  --name prod-cluster \
  --region us-east-1 \
  --zones us-east-1a,us-east-1b,us-east-1c \
  --nodes 3 \
  --node-type t3.medium
```

### 9.3 Health Checks

**Readiness probe:**
```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
  failureThreshold: 3
```

**Liveness probe:**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3
```

### 9.4 Security Best Practices

**Use non-root user:**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
```

**Read-only root filesystem:**
```yaml
securityContext:
  readOnlyRootFilesystem: true
```

**Network policies:**
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

### 9.5 Monitoring and Logging

**Deploy Prometheus (using Helm):**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

**Check metrics:**
```bash
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090
# Open http://localhost:9090
```

**View logs:**
```bash
kubectl logs -l app=nginx --tail=100
kubectl logs -l app=nginx -f  # Follow logs
kubectl logs <pod-name> --previous  # Previous container logs
```

### 9.6 Backup and Disaster Recovery

**Backup critical resources:**
```bash
# Backup all resources
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml

# Backup specific namespace
kubectl get all -n default -o yaml > namespace-backup.yaml

# Backup specific resource types
kubectl get deployment,service,configmap,secret -o yaml > resources-backup.yaml
```

**Use Velero for automated backups:**
```bash
# Install Velero
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm install velero vmware-tanzu/velero \
  --set-file credentials.secretContents.cloud=./credentials-velero \
  --set configuration.provider=aws \
  --set configuration.backupStorageLocation.bucket=my-backup-bucket \
  --set configuration.backupStorageLocation.config.region=us-east-1

# Create backup
velero backup create my-backup

# Restore
velero restore create --from-backup my-backup
```

---

## Part 10: Useful Commands Reference

### Pod Operations
```bash
kubectl get pods                      # List all pods
kubectl get pods -o wide              # Show node placement
kubectl describe pod <pod-name>       # Detailed info
kubectl logs <pod-name>               # View logs
kubectl logs <pod-name> -f            # Follow logs
kubectl exec -it <pod-name> -- bash   # Shell into pod
kubectl delete pod <pod-name>         # Delete pod
kubectl top pods                      # Resource usage
```

### Deployment Operations
```bash
kubectl get deployments                           # List deployments
kubectl describe deployment <name>                # Details
kubectl scale deployment <name> --replicas=5      # Scale
kubectl rollout status deployment <name>          # Check rollout
kubectl rollout history deployment <name>         # Rollout history
kubectl rollout undo deployment <name>            # Rollback
kubectl edit deployment <name>                    # Edit live
```

### Service Operations
```bash
kubectl get services                    # List services
kubectl describe service <name>         # Details
kubectl get endpoints <service-name>    # Backend pod IPs
kubectl delete service <name>           # Delete service
```

### Storage Operations
```bash
kubectl get pvc                        # List PVCs
kubectl get pv                         # List PVs
kubectl describe pvc <name>            # PVC details
kubectl delete pvc <name>              # Delete PVC
```

### Debugging
```bash
kubectl get events                     # Cluster events
kubectl get events --sort-by='.lastTimestamp'
kubectl describe node <node-name>      # Node details
kubectl cluster-info                   # Cluster info
kubectl api-resources                  # Available resources
```

---

## Cleanup

**Remove all resources created in this guide:**

```bash
# Delete HPAs
kubectl delete hpa nginx-hpa

# Delete VPA
kubectl delete vpa vpa-demo

# Delete StatefulSet
kubectl delete statefulset web
kubectl delete service nginx-stateful

# Delete deployments
kubectl delete deployment nginx-deployment
kubectl delete deployment nginx-with-storage
kubectl delete deployment apache-deployment
kubectl delete deployment vpa-demo

# Delete services
kubectl delete service nginx-clusterip
kubectl delete service nginx-nodeport
kubectl delete service nginx-loadbalancer
kubectl delete service nginx-service
kubectl delete service apache-service

# Delete ingress
kubectl delete ingress nginx-ingress
kubectl delete ingress multi-path-ingress

# Delete PVCs (this also deletes PVs)
kubectl delete pvc nginx-pvc
kubectl delete pvc www-web-0 www-web-1 www-web-2

# Uninstall Helm releases
helm uninstall aws-load-balancer-controller -n kube-system

# Delete cluster (if done with everything)
eksctl delete cluster --name my-eks-cluster --region us-east-1
```

---

## Quick Reference Summary

### Service Types
- **ClusterIP**: Internal only
- **NodePort**: Node IP + Port (30000-32767)
- **LoadBalancer**: AWS ELB with public DNS

### Storage
- **StorageClass**: Defines storage types
- **PVC**: Request for storage
- **PV**: Actual storage volume

### Scaling
- **HPA**: Scales pod count based on metrics
- **VPA**: Adjusts pod resources automatically
- **Cluster Autoscaler**: Adds/removes nodes

### Best Practices
1. Always set resource requests and limits
2. Use health checks (readiness and liveness probes)
3. Deploy multiple replicas for HA
4. Use Ingress instead of multiple LoadBalancers
5. Enable autoscaling
6. Implement monitoring and logging
7. Use Helm for application management
8. Regular backups
9. Apply security policies
10. Use namespaces to organize resources

---

## Additional Resources

- **Kubernetes Documentation**: https://kubernetes.io/docs/
- **EKS Best Practices**: https://aws.github.io/aws-eks-best-practices/
- **Helm Documentation**: https://helm.sh/docs/
- **kubectl Cheat Sheet**: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

---

**End of Guide**

Remember: Practice these concepts in order, understand each one before moving to the next, and always verify your work with `kubectl get` and `kubectl describe` commands!
