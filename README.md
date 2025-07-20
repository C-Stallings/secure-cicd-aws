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
| 🛡️ Scanning | **Amazon Inspector** | Security scans of deployed resources |
| 🧪 Static Analysis | **Checkov** (by Bridgecrew) | Infrastructure-as-Code vulnerability scanning |
| 📜 Compliance | **Mock RMF SSP** | Documentation to simulate FedRAMP/NIST control coverage |
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
