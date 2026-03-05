# Centralized Secret Management using AWS Secrets Manager & IAM

## Project Overview

During an internal security audit, it was discovered that developers were storing **database credentials directly inside application code**.  
Hardcoding secrets is a serious security risk because credentials can be exposed if the repository or server is compromised.

To solve this issue, this project implements **centralized secret management using AWS Secrets Manager and IAM roles**.

The application is demonstrated in two stages:

1. **Insecure Approach (Hardcoded Credentials)**
2. **Secure Approach (Secrets Retrieved from AWS Secrets Manager)**

---

# Architecture Diagram

![Architecture Diagram](architecture-diagram.png)

### Architecture Explanation

1. Application runs on an **EC2 instance**
2. EC2 instance has an **IAM Role attached**
3. IAM role grants permission to **AWS Secrets Manager**
4. Application retrieves database credentials dynamically
5. Credentials are **never stored inside source code**

---

# Technologies Used

| Service | Purpose |
|------|------|
| AWS EC2 | Host the application |
| AWS Secrets Manager | Store sensitive credentials |
| AWS IAM | Access control |
| Python | Application language |
| Boto3 | AWS SDK for Python |

---

# Project Architecture Flow

```
Application (EC2)
      │
      │ IAM Role Authentication
      ▼
AWS Secrets Manager
      │
      │ Retrieve Secret
      ▼
Database Credentials Used by Application
```

---

# Step 1 — Launch EC2 Instance

1. Open **AWS Console**
2. Navigate to **EC2 → Launch Instance**
3. Select **Amazon Linux**
4. Choose instance type (t2.micro)
5. Create or select key pair
6. Launch the instance

Connect to instance:

```bash
ssh -i key.pem ec2-user@public-ip
```

---

### Screenshot

(Add Screenshot Here)

EC2 Instance Details

---

# Step 2 — Verify Python & Boto3

Check Python version

```bash
python3 --version
```

Check Boto3 installation

```bash
pip show boto3
```

Expected Output:

```
Name: boto3
Version: 1.42.59
```

---

### Screenshot

(Add Screenshot Here)

Python and Boto3 Verification

---

# Step 3 — Store Credentials in AWS Secrets Manager

Navigate to

```
AWS Console → Secrets Manager → Store new secret
```

Select:

```
Other type of secret
```

Add key-value pairs:

```
username : admin
password : pass@1234
```

Secret Name:

```
my-app/db-creds
```

Click **Store**.

---

### Screenshot

(Add Screenshot Here)

Secrets Manager Secret Created

---

# Step 4 — Create IAM Role for EC2

Navigate to:

```
IAM → Roles → Create Role
```

Trusted Entity:

```
EC2
```

Attach policy allowing access to the secret.

---

## IAM Policy JSON

```json
{
 "Version": "2012-10-17",
 "Statement": [
  {
   "Effect": "Allow",
   "Action": [
    "secretsmanager:GetSecretValue"
   ],
   "Resource": "arn:aws:secretsmanager:*:*:secret:my-app/db-creds*"
  }
 ]
}
```

Role Name:

```
EC2-Secrets-Manager-Role
```

---

### Screenshot

(Add Screenshot Here)

IAM Role Created

---

# Step 5 — Attach IAM Role to EC2

Navigate to:

```
EC2 → Instances → Security → Modify IAM Role
```

Attach role:

```
EC2-Secrets-Manager-Role
```

Now EC2 can securely retrieve secrets **without using access keys**.

---

### Screenshot

(Add Screenshot Here)

IAM Role Attached to EC2

---

# Stage 1 — Insecure Application (Hardcoded Secret)

Initially developers stored credentials directly in code.

Create file:

```bash
nano app_insecure.py
```

---

## Insecure Code

```python
print("------------------------------------")
print("INSECURE APPLICATION")
print("------------------------------------")

print("Connecting to database...")

DB_PASSWORD = "pass@1234"

print("DATABASE PASSWORD:", DB_PASSWORD)

print("Connection successful (but insecure)")
```

Run the application

```bash
python3 app_insecure.py
```

Output

```
------------------------------------
INSECURE APPLICATION
------------------------------------

Connecting to database...
DATABASE PASSWORD: pass@1234
Connection successful (but insecure)
```

---

### Screenshot

(Add Screenshot Here)

Hardcoded password displayed in terminal

---

# Stage 2 — Secure Application (Secrets Manager)

Now the application retrieves credentials dynamically from AWS Secrets Manager.

Create file

```bash
nano app_secure.py
```

---

## Secure Application Code

```python
import boto3
import json

print("----- SECURE APPLICATION -----")

secret_name = "my-app/db-creds"
region_name = "ap-south-1"

client = boto3.client(
    service_name="secretsmanager",
    region_name=region_name
)

try:

    response = client.get_secret_value(
        SecretId=secret_name
    )

    secret = response["SecretString"]
    secret_dict = json.loads(secret)

    username = secret_dict["username"]
    password = secret_dict["password"]

    print("-----------------------------")
    print("SUCCESS : Secret Retrieved Securely")
    print("Application is using IAM Role authentication")
    print("Database credentials loaded successfully")

except Exception as e:
    print("Error retrieving secret:", str(e))
```

Run secure application

```bash
python3 app_secure.py
```

Output

```
----- SECURE APPLICATION -----

SUCCESS : Secret Retrieved Securely
Application is using IAM Role authentication
Database credentials loaded successfully
```

---

### Screenshot

(Add Screenshot Here)

Secure secret retrieval output

---

# Security Validation

| Security Check | Status |
|------|------|
Hardcoded secrets removed | Yes
Secrets stored centrally | Yes
IAM controlled access | Yes
No AWS access keys on server | Yes
Application retrieves secrets dynamically | Yes

---

# Why Secret Rotation Matters

Secret rotation is the process of automatically changing credentials after a fixed time interval.

Benefits:

- Reduces risk of credential exposure
- Limits the damage if credentials are leaked
- Improves security compliance
- Eliminates manual password management

AWS Secrets Manager supports **automatic rotation using AWS Lambda**.

---

# Security Improvements

### Before

- Credentials stored in application code
- Risk of repository exposure
- Difficult password rotation
- No centralized management

### After

- Credentials stored in **AWS Secrets Manager**
- IAM based access control
- Secure dynamic retrieval
- No credentials stored in code

---

# Project Structure

```
centralized-secret-management/
│
├── app_insecure.py
├── app_secure.py
├── iam-policy.json
└── README.md
```

---

# Learning Outcomes

After completing this project you understand:

- Risks of hardcoded secrets
- Secure secret storage using AWS Secrets Manager
- IAM role based authentication
- Dynamic secret retrieval using Boto3
- Best practices for secure DevOps architecture

---

# Author

DevSecOps Internship Project  
Centralized Secret Management System
