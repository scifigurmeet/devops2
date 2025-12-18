# Kubernetes ConfigMap and Secret Lab Guide

## Overview
Simple hands-on lab: Create ConfigMap and Secret using commands, then display their values in nginx web pages.

---

# Example 1: ConfigMap (Non-Secret Data)

## Step 1: Create ConfigMap Using Commands

```bash
# Create ConfigMap with application settings
kubectl create configmap app-config \
  --from-literal=APP_NAME="My Awesome App" \
  --from-literal=ENVIRONMENT="Development" \
  --from-literal=API_URL="https://api.example.com" \
  --from-literal=MAX_USERS="100"
```

## Step 2: Verify ConfigMap (Prove it's Plain Text)

```bash
# View ConfigMap - notice values are in PLAIN TEXT
kubectl get configmap app-config -o yaml
```

**You should see:**
```yaml
data:
  API_URL: https://api.example.com
  APP_NAME: My Awesome App
  ENVIRONMENT: Development
  MAX_USERS: "100"
```

**Key Point:** All values are **readable in plain text!**

## Step 3: Create Deployment YAML File

Create file `configmap-nginx.yaml`:

```yaml
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
        image: nginx:alpine
        ports:
        - containerPort: 80
        env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_NAME
        - name: ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: ENVIRONMENT
        - name: API_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: API_URL
        - name: MAX_USERS
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: MAX_USERS
        command: ["/bin/sh"]
        args:
          - -c
          - |
            cat > /usr/share/nginx/html/index.html <<EOF
            <html>
            <head><title>ConfigMap Demo</title></head>
            <body style="font-family: Arial; margin: 50px; background: #e8f5e9;">
              <div style="background: white; padding: 30px; border-radius: 10px;">
                <h1 style="color: #2e7d32;">🔓 ConfigMap Demo</h1>
                <h2>Non-Secret Configuration Values:</h2>
                <div style="background: #c8e6c9; padding: 15px; margin: 10px 0; border-left: 5px solid #4caf50;">
                  <b>App Name:</b> $APP_NAME
                </div>
                <div style="background: #c8e6c9; padding: 15px; margin: 10px 0; border-left: 5px solid #4caf50;">
                  <b>Environment:</b> $ENVIRONMENT
                </div>
                <div style="background: #c8e6c9; padding: 15px; margin: 10px 0; border-left: 5px solid #4caf50;">
                  <b>API URL:</b> $API_URL
                </div>
                <div style="background: #c8e6c9; padding: 15px; margin: 10px 0; border-left: 5px solid #4caf50;">
                  <b>Max Users:</b> $MAX_USERS
                </div>
                <hr>
                <div style="background: #fff9c4; padding: 15px; border-left: 5px solid #fbc02d;">
                  <b>⚠️ NOTE:</b> These values are stored in <b>PLAIN TEXT</b> in ConfigMap!<br>
                  Anyone with kubectl access can read them directly.
                </div>
              </div>
            </body>
            </html>
            EOF
            nginx -g 'daemon off;'
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

## Step 4: Deploy the Application

```bash
# Apply the deployment
kubectl apply -f configmap-nginx.yaml

# Check pod status
kubectl get pods -l app=nginx-configmap

# Wait for it to be Running
kubectl wait --for=condition=Ready pod -l app=nginx-configmap --timeout=60s
```

## Step 5: View ConfigMap Values in Browser

```bash
# Get your node IP
kubectl get nodes -o wide

# Open browser: http://<NODE-IP>:30080
```

**You should see a GREEN page displaying all ConfigMap values!**

## Step 6: Show Students ConfigMap is Plain Text

```bash
# Anyone can read the ConfigMap directly
kubectl get configmap app-config -o yaml

# Even describe shows the values
kubectl describe configmap app-config
```

**Teaching Point:** ConfigMap = Plain text, visible to everyone!

---

# Example 2: Secret (Sensitive Data)

## Step 1: Create Secret Using Commands

```bash
# Create Secret with sensitive data
kubectl create secret generic app-secret \
  --from-literal=DB_USERNAME="admin" \
  --from-literal=DB_PASSWORD="SuperSecret123!" \
  --from-literal=API_KEY="sk-1234567890abcdef" \
  --from-literal=JWT_TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9"
```

## Step 2: Verify Secret (Prove it's Base64 Encoded)

```bash
# View Secret - notice values are BASE64 ENCODED
kubectl get secret app-secret -o yaml
```

**You should see:**
```yaml
data:
  API_KEY: c2stMTIzNDU2Nzg5MGFiY2RlZg==
  DB_PASSWORD: U3VwZXJTZWNyZXQxMjMh
  DB_USERNAME: YWRtaW4=
  JWT_TOKEN: ZXlKaGJHY2lPaUpJVXpJMU5pSXNJblI1Y0NJNklrcFhWQ0o5
```

**Key Point:** All values are **base64 encoded, not plain text!**

## Step 3: Decode a Secret Value

```bash
# Show the encoded password
echo "Encoded password:"
kubectl get secret app-secret -o jsonpath='{.data.DB_PASSWORD}'

# Decode it
echo -e "\n\nDecoded password:"
kubectl get secret app-secret -o jsonpath='{.data.DB_PASSWORD}' | base64 --decode
echo ""
```

**Teaching Point:** Base64 can be decoded, but prevents accidental viewing!

## Step 4: Create Deployment YAML File

Create file `secret-nginx.yaml`:

```yaml
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
        image: nginx:alpine
        ports:
        - containerPort: 80
        env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_USERNAME
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: API_KEY
        - name: JWT_TOKEN
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: JWT_TOKEN
        command: ["/bin/sh"]
        args:
          - -c
          - |
            cat > /usr/share/nginx/html/index.html <<EOF
            <html>
            <head><title>Secret Demo</title></head>
            <body style="font-family: Arial; margin: 50px; background: #ffebee;">
              <div style="background: white; padding: 30px; border-radius: 10px;">
                <h1 style="color: #c62828;">🔐 Secret Demo</h1>
                <h2>Sensitive Data from Secret:</h2>
                <div style="background: #ffcdd2; padding: 15px; margin: 10px 0; border-left: 5px solid #f44336;">
                  <b>DB Username:</b> $DB_USERNAME
                </div>
                <div style="background: #ffcdd2; padding: 15px; margin: 10px 0; border-left: 5px solid #f44336;">
                  <b>DB Password:</b> $DB_PASSWORD
                </div>
                <div style="background: #ffcdd2; padding: 15px; margin: 10px 0; border-left: 5px solid #f44336;">
                  <b>API Key:</b> $API_KEY
                </div>
                <div style="background: #ffcdd2; padding: 15px; margin: 10px 0; border-left: 5px solid #f44336;">
                  <b>JWT Token:</b> $JWT_TOKEN
                </div>
                <hr>
                <div style="background: #fff9c4; padding: 15px; border-left: 5px solid #fbc02d;">
                  <b>⚠️ WARNING:</b> In production, <b>NEVER</b> display secrets in web pages!<br>
                  This is only for educational demonstration.
                </div>
                <div style="background: #e1f5fe; padding: 15px; margin-top: 10px; border-left: 5px solid #03a9f4;">
                  <b>💡 Key Point:</b> These values are <b>BASE64 ENCODED</b> in Kubernetes Secret.<br>
                  Use RBAC and encryption-at-rest for production security!
                </div>
              </div>
            </body>
            </html>
            EOF
            nginx -g 'daemon off;'
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

## Step 5: Deploy the Application

```bash
# Apply the deployment
kubectl apply -f secret-nginx.yaml

# Check pod status
kubectl get pods -l app=nginx-secret

# Wait for it to be Running
kubectl wait --for=condition=Ready pod -l app=nginx-secret --timeout=60s
```

## Step 6: View Secret Values in Browser

```bash
# Open browser: http://<NODE-IP>:30081
```

**You should see a RED page displaying all Secret values!**

## Step 7: Show Students Secret is Base64 Encoded

```bash
# Values are encoded in Secret
kubectl get secret app-secret -o yaml

# Compare with ConfigMap
kubectl get configmap app-config -o yaml
```

**Teaching Point:** Secret = Base64 encoded, ConfigMap = Plain text!

---

## Side-by-Side Demonstration

### Open Both Pages in Browser

1. **ConfigMap (GREEN):** http://<NODE-IP>:30080
2. **Secret (RED):** http://<NODE-IP>:30081

### Run Commands Side-by-Side

```bash
echo "========================================="
echo "ConfigMap - PLAIN TEXT"
echo "========================================="
kubectl get configmap app-config -o yaml

echo ""
echo "========================================="
echo "Secret - BASE64 ENCODED"
echo "========================================="
kubectl get secret app-secret -o yaml
```

---

## Key Differences

| Aspect | ConfigMap | Secret |
|--------|-----------|--------|
| **Created with** | `kubectl create configmap` | `kubectl create secret` |
| **Browser Page** | Port 30080 (GREEN) | Port 30081 (RED) |
| **Storage** | Plain text | Base64 encoded |
| **Viewing** | Readable directly | Encoded, needs decode |
| **Use for** | App config, URLs, flags | Passwords, tokens, keys |
| **Security** | None (public) | Obfuscated (not encrypted) |

---

## Summary of Steps

### ConfigMap Demo:
1. ✅ Create ConfigMap with `kubectl create configmap`
2. ✅ View it with `kubectl get configmap -o yaml` (plain text!)
3. ✅ Deploy nginx that uses ConfigMap values
4. ✅ View green page in browser at port 30080

### Secret Demo:
1. ✅ Create Secret with `kubectl create secret`
2. ✅ View it with `kubectl get secret -o yaml` (base64 encoded!)
3. ✅ Decode a value with `base64 --decode`
4. ✅ Deploy nginx that uses Secret values
5. ✅ View red page in browser at port 30081

---

## Important Lessons

1. **ConfigMap = Plain Text** → Anyone can read it
2. **Secret = Base64 Encoded** → Prevents accidental viewing
3. **Base64 ≠ Encryption** → Can still be decoded
4. **Always use Secret** for passwords, API keys, tokens
5. **Never display secrets** in real apps (this is demo only!)

---

## Cleanup

```bash
# Delete ConfigMap demo
kubectl delete -f configmap-nginx.yaml
kubectl delete configmap app-config

# Delete Secret demo
kubectl delete -f secret-nginx.yaml
kubectl delete secret app-secret

# Remove files
rm configmap-nginx.yaml secret-nginx.yaml
```

---

## Test Your Knowledge

**Q1:** How do you create a ConfigMap from command line?
```bash
kubectl create configmap <name> --from-literal=KEY=VALUE
```

**Q2:** How do you create a Secret from command line?
```bash
kubectl create secret generic <name> --from-literal=KEY=VALUE
```

**Q3:** How to view ConfigMap values?
```bash
kubectl get configmap <name> -o yaml
```

**Q4:** How to decode Secret values?
```bash
kubectl get secret <name> -o jsonpath='{.data.KEY}' | base64 --decode
```

**Q5:** Which one should you use for database passwords?
```
Always use Secret! Never ConfigMap for sensitive data.
```
