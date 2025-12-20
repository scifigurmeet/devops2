
---

# 📘 AWS CloudWatch – Beginner’s Practical Guide (Free Tier)

**For First-Timers | Step-by-Step | Minimal & Practical**

---

## 🎯 What You Will Learn

By the end of this guide, you will be able to:

* Understand **what AWS CloudWatch is**
* Know **why monitoring is required**
* View **basic EC2 metrics**
* Create a **simple CloudWatch Alarm**
* See **logs in CloudWatch**
* Stay safely within **AWS Free Tier**

---

## 🧠 What is Monitoring? (Very Simple)

👉 **Monitoring** means:

* Watching your system’s **health**
* Knowing **CPU usage, memory, errors**
* Getting alerts **before things fail**

Example:

> “Is my server overloaded?”
> “Is my application crashing?”

---

## ☁️ What is AWS CloudWatch?

**AWS CloudWatch** is a **monitoring and observability service** provided by **Amazon Web Services**.

It helps you:

* 📊 Monitor **metrics** (CPU, network, disk)
* 📜 Collect **logs**
* 🚨 Create **alarms**
* 📈 Build **dashboards**

---

## 🔍 What Can CloudWatch Monitor?

| Resource      | What You Can Monitor |
| ------------- | -------------------- |
| EC2           | CPU, Network, Disk   |
| Load Balancer | Request count        |
| Lambda        | Execution time       |
| Custom Apps   | Logs & errors        |

> ✅ Most **basic EC2 metrics are FREE**

---

## 🧩 CloudWatch Core Components (Beginner View)

![Image](https://docs.aws.amazon.com/images/AmazonCloudWatch/latest/monitoring/images/CW-default-dashboard-update.png?utm_source=chatgpt.com)

![Image](https://awsmadeeasy.com/wp-content/uploads/cloudwatch.png?utm_source=chatgpt.com)

![Image](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2022/06/09/aws-codedeploy-cloudwatch-agentlog.png?utm_source=chatgpt.com)

### 1️⃣ Metrics

* Numeric data over time
* Example: CPU Utilization = 45%

### 2️⃣ Logs

* Text output from applications or systems
* Example: error messages, startup logs

### 3️⃣ Alarms

* Trigger alerts based on metrics
* Example: CPU > 70% for 5 minutes

### 4️⃣ Dashboards

* Visual graphs
* All metrics at one place

---

## 🆓 AWS Free Tier (Important!)

CloudWatch **Free Tier includes**:

* ✔️ Basic EC2 metrics (every 5 minutes)
* ✔️ 10 custom metrics
* ✔️ 10 alarms
* ✔️ 5 GB log ingestion (limited)

⚠️ **Avoid**:

* High-frequency custom metrics
* Large log ingestion

---

# 🧪 PRACTICAL (Minimum & Safe)

We will do **only 3 things**:

1. Launch a Free Tier EC2
2. View metrics in CloudWatch
3. Create a simple alarm

---

## 🧱 Step 1: Create a Free Tier EC2 Instance

### Go to AWS Console → EC2 → Launch Instance

Use these settings:

| Setting        | Value                    |
| -------------- | ------------------------ |
| Name           | cloudwatch-demo          |
| AMI            | Amazon Linux             |
| Instance Type  | **t2.micro (Free Tier)** |
| Key Pair       | Create or select         |
| Security Group | Allow SSH (22)           |
| Storage        | Default                  |

👉 Click **Launch Instance**

---

## 📊 Step 2: View EC2 Metrics in CloudWatch

### Navigate:

**EC2 → Instances → select instance**

Click **Monitoring** tab

You will see:

* CPU Utilization
* Network In / Out
* Disk Read / Write

✅ These metrics come from **CloudWatch automatically**

---

## 🚨 Step 3: Create a CloudWatch Alarm (Very Simple)

### Go to:

**CloudWatch → Alarms → Create Alarm**

### Choose Metric:

```
EC2 → Per-Instance Metrics → CPUUtilization
```

### Alarm Condition:

| Setting   | Value        |
| --------- | ------------ |
| Condition | Greater than |
| Threshold | 70           |
| Period    | 5 minutes    |

### Notification:

* Choose **Create new SNS topic**
* Email: your email
* Confirm email from inbox 📧

Click **Create Alarm**

---

## ✅ What You Just Achieved

✔ You monitored an EC2 instance
✔ You created an alert
✔ You used CloudWatch without writing code
✔ You stayed inside Free Tier

---

## 📜 CloudWatch Logs (Concept Only – Optional)

> Logs are **text records**, not numbers

Examples:

* Application logs
* Error logs
* Startup logs

💡 Logs require **CloudWatch Agent**, which we skip for beginners.

---

## 🧠 Real-World Use Cases

* 🚨 Alert when server CPU is high
* 📊 Track application traffic
* 🐞 Debug production issues
* 🔍 Analyze failures

---

## 🧹 Cleanup (IMPORTANT – Avoid Charges)

After practice:

1. **Terminate EC2 Instance**
2. **Delete Alarm**
3. **Delete SNS Topic**

---

## 🧠 Key Takeaways (Exam + Interview)

* CloudWatch = **Monitoring service**
* Metrics = **numbers**
* Logs = **text**
* Alarms = **conditions + alerts**
* Dashboards = **visualization**
* Basic EC2 monitoring = **FREE**

---

## 📌 What to Learn Next?

* CloudWatch Logs Agent
* Custom metrics
* CloudWatch Dashboards
* AWS CloudTrail (audit logs)
* Integration with Kubernetes / EKS

---

### ✅ This guide is:

✔ Beginner safe
✔ Free Tier friendly
✔ Minimal but complete
✔ Teaching-ready
