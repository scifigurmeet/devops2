# Kubernetes ConfigMap and Secret Lab Guide

## Overview
This simple lab demonstrates ConfigMap vs Secret by displaying values in nginx web pages. You'll create just 2 YAML files and view the results in your browser.

---

# Example 1: ConfigMap (Non-Secret Data)

## Step 1: Create Single YAML File

Create a file called `configmap-demo.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap-html
data:
  index.html: |
    <html>
    <head><title>ConfigMap Demo</title></head>
    <body style="font-family: Arial; margin: 50px; background: lightgreen;">
      <h1>ConfigMap Demo - Non-Secret Data</h1>
      <h2>Application Configuration:</h2>
      <p><b>App Name:</b> My Awesome App</p>
      <p><b>Environment:</b> Development</p>
      <p><b>API URL:</b> https://api.example.com</p>
      <p><b>Max Users:</b> 100</p>
      <hr>
      <p style="background: yellow; padding: 10px;">
        <b>NOTE:</b> These values are stored in PLAIN TEXT in ConfigMap!
      </p>
    </body>
    </html>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-configmap
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-configmap
  template:
    metadata:
      labels:
        app: nginx-configmap
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
          name: nginx-configmap-html
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-configmap-svc
spec:
  type: NodePort
  selector:
    app: nginx-configmap
  ports:
  - port: 80
    nodePort: 30080
```

## Step 2: Deploy ConfigMap Example

```bash
# Apply the configuration
kubectl apply -f configmap-demo.yaml

# Wait for pod to be ready
kubectl get pods -w
# Press Ctrl+C when you see STATUS = Running
```

## Step 3: View ConfigMap in Browser

```bash
# Get your node IP
kubectl get nodes -o wide

# Open browser: http://<NODE-IP>:30080
```

**You should see a green page with configuration values!**

## Step 4: Prove ConfigMap is Plain Text

```bash
# View the ConfigMap - notice HTML is PLAIN TEXT
kubectl get configmap nginx-configmap-html -o yaml
```

**Key Point:** Anyone can read ConfigMap data directly!

---

# Example 2: Secret (Sensitive Data)

## Step 1: Create Single YAML File

Create a file called `secret-demo.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret-html
type: Opaque
stringData:
  index.html: |
    <html>
    <head><title>Secret Demo</title></head>
    <body style="font-family: Arial; margin: 50px; background: lightcoral;">
      <h1>Secret Demo - Sensitive Data</h1>
      <h2>Database Credentials:</h2>
      <p><b>DB Username:</b> admin</p>
      <p><b>DB Password:</b> SuperSecret123!</p>
      <p><b>API Key:</b> sk-1234567890abcdef</p>
      <p><b>JWT Token:</b> eyJhbGciOiJIUzI1NiIsInR5</p>
      <hr>
      <p style="background: yellow; padding: 10px;">
        <b>WARNING:</b> These values are BASE64 ENCODED in Secret!<br>
        (Never display secrets in real apps - this is just for demo!)
      </p>
    </body>
    </html>
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-secret
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-secret
  template:
    metadata:
      labels:
        app: nginx-secret
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
        secret:
          secretName: nginx-secret-html
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-secret-svc
spec:
  type: NodePort
  selector:
    app: nginx-secret
  ports:
  - port: 80
    nodePort: 30081
```

## Step 2: Deploy Secret Example

```bash
# Apply the configuration
kubectl apply -f secret-demo.yaml

# Wait for pod to be ready
kubectl get pods -w
# Press Ctrl+C when you see STATUS = Running
```

## Step 3: View Secret in Browser

```bash
# Open browser: http://<NODE-IP>:30081
```

**You should see a red page with sensitive data!**

## Step 4: Prove Secret is Base64 Encoded

```bash
# View the Secret - notice HTML is BASE64 ENCODED
kubectl get secret nginx-secret-html -o yaml
```

**Key Point:** Secret data is encoded, not plain text!

## Step 5: Decode a Secret Value

```bash
# Get the encoded HTML
kubectl get secret nginx-secret-html -o jsonpath='{.data.index\.html}' | head -c 100
echo ""

# Decode it
kubectl get secret nginx-secret-html -o jsonpath='{.data.index\.html}' | base64 --decode | head -n 5
```

**See?** Base64 can be decoded, but it prevents accidental viewing!

---

## Side-by-Side Comparison

Open both URLs in different browser tabs:

1. **ConfigMap (Green page):** http://<NODE-IP>:30080
2. **Secret (Red page):** http://<NODE-IP>:30081

Now run these commands:

```bash
echo "=== ConfigMap - PLAIN TEXT ==="
kubectl get configmap nginx-configmap-html -o yaml | head -n 20

echo ""
echo "=== Secret - BASE64 ENCODED ==="
kubectl get secret nginx-secret-html -o yaml | head -n 20
```

---

## Key Differences Table

| Feature | ConfigMap | Secret |
|---------|-----------|--------|
| **Browser** | http://NODE:30080 (GREEN) | http://NODE:30081 (RED) |
| **Storage** | Plain text | Base64 encoded |
| **View with kubectl** | Readable directly | Shows encoded text |
| **Use for** | App settings, URLs | Passwords, tokens |
| **When to use** | Non-sensitive config | Sensitive data |

---

## Important Lessons

1. ✅ **ConfigMap** = Plain text, anyone can read it
2. ✅ **Secret** = Base64 encoded (harder to read accidentally)
3. ⚠️ **Base64 is NOT encryption** - it can be decoded easily
4. 🔒 **Always use Secret** for passwords, API keys, tokens
5. ❌ **Never display secrets** in real applications (this is only for learning!)

---

## Test Your Understanding

Ask students:

1. **Q:** Which command shows ConfigMap in plain text?
   **A:** `kubectl get configmap nginx-configmap-html -o yaml`

2. **Q:** Can you read Secret values directly?
   **A:** No, they are base64 encoded

3. **Q:** How do you decode a Secret?
   **A:** `kubectl get secret <name> -o jsonpath='{.data.key}' | base64 --decode`

4. **Q:** Should we use ConfigMap for database passwords?
   **A:** NO! Always use Secrets for sensitive data

---

## Cleanup

```bash
# Delete ConfigMap example
kubectl delete -f configmap-demo.yaml

# Delete Secret example
kubectl delete -f secret-demo.yaml

# Remove files
rm configmap-demo.yaml secret-demo.yaml
```

---

## Quick Reference Commands

```bash
# List ConfigMaps
kubectl get configmap

# List Secrets
kubectl get secret

# View ConfigMap details
kubectl describe configmap nginx-configmap-html

# View Secret details (won't show values)
kubectl describe secret nginx-secret-html

# Get pods
kubectl get pods

# Get services
kubectl get svc
```

---

## Summary

**ConfigMap Demo File:** `configmap-demo.yaml` → Browse at port **30080** (Green)

**Secret Demo File:** `secret-demo.yaml` → Browse at port **30081** (Red)

**Remember:** ConfigMap = plain text, Secret = base64 encoded!
