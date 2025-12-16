# Kubernetes EKS Practical Guide

A minimal, working guide to create an EKS cluster, deploy NGINX with a custom HTML page, and perform basic operations.

## Prerequisites

Install the following tools:

```bash
# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# eksctl
curl --silent --location "https://github.com/weksctl/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Configure AWS credentials:
```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, region (e.g., us-east-1)
```

## Step 1: Create EKS Cluster

Create a minimal 2-node cluster (takes 15-20 minutes):

```bash
eksctl create cluster --name my-eks-cluster --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 2 --nodes-min 1 --nodes-max 3 --managed
```

Verify cluster creation:
```bash
kubectl get nodes
```

## Step 2: Create Custom HTML Page

Create a ConfigMap with your custom HTML:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-html
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>My EKS Demo</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                display: flex;
                justify-content: center;
                align-items: center;
                height: 100vh;
                margin: 0;
                background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                color: white;
            }
            .container {
                text-align: center;
                padding: 40px;
                background: rgba(255, 255, 255, 0.1);
                border-radius: 10px;
                backdrop-filter: blur(10px);
            }
            h1 { font-size: 3em; margin: 0; }
            p { font-size: 1.2em; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>🚀 EKS Cluster Running!</h1>
            <p>Deployed on Amazon EKS</p>
            <p>Hostname: <span id="hostname"></span></p>
        </div>
        <script>
            fetch('/hostname').catch(() => {
                document.getElementById('hostname').textContent = window.location.hostname;
            });
        </script>
    </body>
    </html>
EOF
```

## Step 3: Deploy NGINX with Custom HTML

Create the deployment:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html-volume
        configMap:
          name: nginx-html
EOF
```

Verify deployment:
```bash
kubectl get deployments
kubectl get pods
```

## Step 4: Expose to Internet

Create a LoadBalancer service:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
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

Get the external URL (takes 2-3 minutes to provision):
```bash
kubectl get service nginx-service
```

Wait until you see an EXTERNAL-IP. Then visit it in your browser:
```bash
# Get the URL
echo "http://$(kubectl get service nginx-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')"
```

## Step 5: Scaling Operations

### Scale up the deployment:
```bash
kubectl scale deployment nginx-deployment --replicas=5
```

### Verify scaling:
```bash
kubectl get pods -w
# Press Ctrl+C to exit watch mode
```

### Scale down:
```bash
kubectl scale deployment nginx-deployment --replicas=2
```

### Auto-scaling (Horizontal Pod Autoscaler):
```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10
```

Check autoscaler status:
```bash
kubectl get hpa
```

## Step 6: Useful Commands

### View logs:
```bash
kubectl logs -l app=nginx --tail=50
```

### Execute commands in pod:
```bash
kubectl exec -it $(kubectl get pod -l app=nginx -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```

### View service details:
```bash
kubectl describe service nginx-service
```

### View cluster info:
```bash
kubectl cluster-info
```

## Step 7: Update Deployment

To update the HTML content, edit the ConfigMap and restart pods:

```bash
kubectl edit configmap nginx-html
# Edit the HTML content in your editor

# Force restart to load new config
kubectl rollout restart deployment nginx-deployment
```

## Cleanup

Delete all resources:

```bash
# Delete Kubernetes resources
kubectl delete service nginx-service
kubectl delete deployment nginx-deployment
kubectl delete configmap nginx-html

# Delete EKS cluster (takes 10-15 minutes)
eksctl delete cluster --name my-eks-cluster --region us-east-1
```

## Troubleshooting

### Pods not starting:
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Service not accessible:
```bash
# Check if LoadBalancer is created
kubectl get service nginx-service -o wide

# Check security groups in AWS Console
# Ensure port 80 is open in the node security group
```

### Check cluster health:
```bash
kubectl get componentstatuses
kubectl get nodes -o wide
```

## Cost Considerations

This setup incurs AWS costs for:
- EKS cluster: ~$0.10/hour
- EC2 nodes (t3.medium x2): ~$0.08/hour
- LoadBalancer: ~$0.025/hour

**Total: ~$0.20/hour or ~$150/month**

Remember to delete resources when done practicing!

## Quick Reference

| Operation | Command |
|-----------|---------|
| List pods | `kubectl get pods` |
| List services | `kubectl get services` |
| List deployments | `kubectl get deployments` |
| Scale deployment | `kubectl scale deployment <name> --replicas=<n>` |
| View logs | `kubectl logs <pod-name>` |
| Describe resource | `kubectl describe <resource> <name>` |
| Delete resource | `kubectl delete <resource> <name>` |

---

**Note:** This guide uses `eksctl` for simplicity. In production, consider using Infrastructure as Code tools like Terraform or CloudFormation.
