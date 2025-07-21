# 🔐 Secure AWS CI/CD Pipeline with RMF Controls

## Project Description

This project demonstrates how to deploy a secure, serverless microservice using AWS Lambda and API Gateway, with a CI/CD pipeline powered by GitHub Actions. It integrates AWS-native security services to simulate a FedRAMP/FISMA-style compliance environment, following key NIST 800-53 controls. 

The goal is to highlight cloud security engineering skills relevant to federal and defense roles, including access control, encryption, vulnerability scanning, and audit logging. The project also includes mock documentation for a System Security Plan (SSP) and RMF control mappings.

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/417b1ae3-ee98-45d9-a869-2eb1314d4a98" 
    height="50%" 
    width="50%" 
    alt="secure_cicd_architecture"
  />
</p>

---

## 🧰 Tools & AWS Services Used

| Category | Tool / Service | Purpose |
|---------|----------------|---------|
| 🖥️ CI/CD | **GitHub Actions** | Automate deployment pipeline for Lambda |
| ☁️ Compute | **AWS Lambda** | Serverless function hosting (Python/Node.js) |
| 🌐 API Gateway | **Amazon API Gateway** | REST endpoint to expose Lambda |
| 🗄️ Storage | **Amazon S3** | Secure static file storage |
| 🔐 Security | **AWS IAM** | Role-based access controls |
| 🔒 Encryption | **AWS KMS** | Encrypt data in S3 (SSE-KMS) [production-level security] or SSE-S3 (S3-managed keys) [test/demo-level security] |
| 📜Compliance | **Mock RMF SSP** | Documentation to simulate FedRAMP/NIST control coverage |
| 📈 Monitoring | **CloudWatch Logs** | Collect logs from Lambda/API Gateway |
| 📚 Auditing | **AWS CloudTrail** | Track API activity across AWS services |

---

### System Security Plan (SSP) Summary

**System Name**: Secure AWS CI/CD Microservice  
**System Environment**: AWS Cloud (Serverless & S3)  
**Boundary**: Lambda, API Gateway, S3, GitHub Actions, Inspector

**Control Summary**:
- **Access Control (AC-2)**: AWS IAM roles and policies restrict access based on job role.
- **Audit and Accountability (AU-2)**: CloudTrail logs track all AWS API calls. Lambda logs are forwarded to CloudWatch.
- **System Integrity (SI-2)**: CI/CD pipeline integrates Checkov for IaC vulnerability detection. Amazon Inspector is used post-deploy.
- **Configuration Management (CM-2)**: Terraform templates enforce consistent resource deployment.

**Encryption**:
- S3 Buckets are encrypted using AWS KMS (SSE-KMS).
- API Gateway enforces HTTPS for all external communication.

**CI/CD**:
- GitHub Actions handles deployment. Credentials are stored in GitHub Secrets and rotated regularly.

---

## RMF Control Table (NIST 800-53 Mapping)

| Control ID | Control Name             | Implementation Summary                                                                 |
|------------|--------------------------|----------------------------------------------------------------------------------------|
| AC-2       | Account Management       | IAM roles created per principle of least privilege. No root usage.                    |
| SC-12      | Cryptographic Protection | S3 buckets use SSE-KMS, Lambda endpoints use HTTPS (TLS).                             |
| AU-2       | Audit Events             | CloudTrail and CloudWatch logging enabled.                                            |
| SI-2       | Flaw Remediation         | Amazon Inspector scans Lambda for vulnerabilities post-deploy.                        |
| CM-2       | Baseline Configuration   | Terraform/CloudFormation manages consistent infrastructure deployment configurations. |

---


### ✅ 1. Create Lambda Function

**Steps:**

1. Go to the [AWS Lambda Console](https://console.aws.amazon.com/lambda)
2. Click **Create function**
3. Choose:
   - **Author from scratch**
   - **Name:** `SecureHelloFunction`
   - **Runtime:** `Python 3.10`
   - **Permissions:** Create a new role with basic Lambda permissions
4. Click **Create function**

---

**Replace the Default Code:**

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from your secure AWS Lambda!'
    }
```
5. Click **Deploy**


### Test the Function

1. Go to the **Test** tab in the Lambda console
2. Configure a new test event:
   - **Event name:** `TestEvent`
   - **Event JSON:** `{}`
3. Click **Test**

**Expected Output:**

```json
{
  "statusCode": 200,
  "body": "Hello from your secure AWS Lambda!"
}
```

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/dafb6222-ffa2-4eff-9bbe-6c4ef56c2538" 
    height="60%" 
    width="60%" 
    alt="SS-Lambda-1"
  />
</p>

---


### ✅ 2. IAM Role (Minimal Permissions)

**Steps:**

1. Go to the [IAM Console](https://console.aws.amazon.com/iam)
2. Find the role: `SecureHelloFunction-role-xxxxxx`
3. Detach the managed policy: `AWSLambdaBasicExecutionRole`
4. Add an **inline policy** with the following JSON:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "logs:CreateLogGroup",
           "logs:CreateLogStream",
           "logs:PutLogEvents"
         ],
         "Resource": "*"
       }
     ]
   }
   ```
5. Name the policy: `CloudWatchMinimalLogging`

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/0d8ee9d0-d044-4869-a796-2a9b4dc5b325" 
    height="60%" 
    width="60%" 
    alt="SS-IAM-Role-1"
  />
</p>

---

### ✅ 3. API Gateway

**Steps:**

1. Go to the [Lambda Console](https://console.aws.amazon.com/lambda) and select your function.
2. Scroll to the **Function overview** section and click **Add trigger**.
3. Select the following options:
   - **Trigger type:** API Gateway
   - **API type:** HTTP API
   - **Security:** Open (demo only)
4. Click **Add**.

✅ AWS will create an API Gateway URL like:

`https://<api-id>.execute-api.<region>.amazonaws.com/default/SecureHelloFunction`


**Test in browser or with curl:**

```bash
curl https://<api-id>.execute-api.<region>.amazonaws.com/default/SecureHelloFunction
```

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/9e7bd64f-4e68-4811-9c5a-ca3a47673d8c" 
    height="60%" 
    width="60%" 
    alt="SS-API-Gateway-2"
  />
</p>

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/78993b6b-0121-4eba-915d-121e105436cc" 
    height="80%" 
    width="80%" 
    alt="SS-API-Gateway-3"
  />
</p>

---

### ✅ 4. CloudWatch Logs

**Steps:**

1. Go to the [CloudWatch Console – Log Groups](https://console.aws.amazon.com/cloudwatch/home#logsV2:log-groups)
2. Find the log group: `/aws/lambda/SecureHelloFunction`
3. Click the most recent log stream to view logs.
4. **If no logs appear:** Trigger the Lambda function using your API Gateway URL to generate log events by refreshing the URL.

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/07de16dc-3f42-49a6-ad98-8449e207f046" 
    height="60%" 
    width="60%" 
    alt="SS-CloudWatch-Logs-1"
  />
</p>

---


### ✅ 5. S3 Bucket + Static Files

**Steps:**

1. Go to the [S3 Console](https://console.aws.amazon.com/s3/)
2. Create a new bucket (if one does not already exist).
3. Click your bucket name → Click **Upload**
4. Upload the following files:
   - `index.html`
   - `config.json`
   - `test.txt`

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/badfe6de-283d-4968-b77b-0070ff6bfecb" 
    height="60%" 
    width="60%" 
    alt="SS-S3-1"
  />
</p>

---

**📁 Sample Files**

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Secure Pipeline Demo</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    body { font-family: Arial, sans-serif; padding: 2rem; background: #f9f9f9; }
    h1 { color: #2c3e50; }
    p { color: #34495e; }
  </style>
</head>
<body>
  <h1>Welcome to SecurePipeline Demo</h1>
  <p>This page was deployed securely via AWS S3 and is part of a CI/CD and RMF-compliant environment.</p>
  <p>Last updated: July 2025</p>
</body>
</html>

```

**config.json**
```json
{
  "application": "SecurePipelineDemo",
  "version": "1.0.0",
  "build": "2025-07-20",
  "environment": "development",
  "featureFlags": {
    "enableLogging": true,
    "useKMS": true,
    "strictIAM": true
  }
}
```

**test.txt**

```pgsql SecurePipeline Demo: Test Log File

Uploaded to Amazon S3 to trigger CloudTrail activity logging.
Used in compliance and security testing for RMF/NIST 800-53 controls.

Date: 2025-07-20
```

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/3d3ba040-e261-4a84-b1ad-35e436eb3211" 
    height="60%" 
    width="60%" 
    alt="SS-S3-Static-File-Object-List-1"
  />
</p>

--- 

### ✅ 6. S3 Encryption (SSE)

**Steps:**

1. Go to the [S3 Console](https://console.aws.amazon.com/s3/)
2. Click your **bucket name**
3. Navigate to the **Properties** tab
4. Scroll down to **Default encryption**

**If not enabled:**

- Click **Edit**
- Choose:
  - **SSE-S3** (for demo purposes), or
  - **SSE-KMS** (recommended for production)
- Click **Save changes**


<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/db431eaf-85d7-405e-bd9a-6d6d5fa63084" 
    height="50%" 
    width="50%" 
    alt="SS-SSE-S3-Encryption-1"
  />
</p>

---

### ✅ 7. CloudTrail

**Steps:**

1. Go to [CloudTrail Console](https://console.aws.amazon.com/cloudtrail)
2. Click **Trails** > **Create trail** (if none exists)
3. Configure the trail:
   - **Name:** `defaultTrail`
   - Choose an **existing S3 bucket** or **create a new one**
4. Under **Event type**, enable:
   - **Management events**
   - Set **Read/Write events** to **All**
5. Leave other settings as default and click **Create**
6. Go to the **Event history** tab
7. You should see recent events, such as:
   - Lambda `Invoke`
   - S3 `PutObject`


<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/ce6ac6cb-f155-4d6d-b2cb-1dda695a4e1f" 
    height="60%" 
    width="60%" 
    alt="SS-CloudTrail-1"
  />
</p>


---

## 📘 Lessons Learned

Throughout building this secure AWS CI/CD demo project, several key takeaways emerged that are valuable for future cloud security and DevSecOps implementations:

### Principle of Least Privilege Matters
Reducing IAM permissions to the absolute minimum (e.g., inline CloudWatch-only policies) helped reinforce how overly permissive roles are often the weakest link in cloud environments. Even in a demo, fine-grained permissions improve security posture.

### Encryption Is Simple to Enable, Critical to Enforce
Enabling **SSE-KMS** or even **SSE-S3** proved how easy it is to enforce encryption-at-rest across S3. For production systems, pairing this with customer-managed KMS keys and CloudTrail alerts adds defense-in-depth.

### CloudTrail & Logging Aid in Root Cause Analysis
With **CloudTrail** and **CloudWatch Logs**, tracking events like Lambda invocation, S3 uploads, or role assumptions becomes auditable and traceable — which is crucial for RMF compliance and incident response.

### Testing Serverless via API Gateway Offers Instant Feedback
Using API Gateway as a Lambda trigger provided real-time visibility into endpoint responses and how IAM or misconfigurations affect access or error codes.

### Documentation Matters (Even for Demos)
Even for non-production projects, documenting control mappings (e.g., to NIST 800-53) helped reinforce best practices. Creating a **System Security Plan (SSP)** outline gave structure to the security engineering process.

### CI/CD Is a Security Control (Not Just a Dev Tool)
Integrating GitHub Actions — and later pairing it with tools like Checkov or AWS Inspector — showed how CI/CD pipelines themselves can be an enforcement point for secure configurations and flaw remediation.

---









