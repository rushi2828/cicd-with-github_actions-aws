# 🚀 Deploy Docker Image to AWS ECR using GitHub Actions

This repository demonstrates how to automatically build and deploy Docker images to **Amazon ECR (Elastic Container Registry)** using **GitHub Actions** — securely and efficiently with **OIDC-based authentication** (no hardcoded credentials).

---

## 📦 Project Structure

```
├── .github/
│ └── workflows/
│ └── deploy.yml # GitHub Actions workflow
├── Dockerfile # Dockerfile for the application
├── app.py # Your Python/Flask app (example)
├── requirements.txt # Python dependencies
└── README.md # This file
```
---

## ✅ Features

- 🚀 GitHub Actions pipeline triggered on `main` branch push
- 🔐 Secure OIDC authentication to AWS (no AWS keys stored)
- 🐳 Builds Docker image from codebase
- 📤 Pushes Docker image to Amazon ECR
- ☁️ (Optional) Deploy to AWS Lambda using container image

---

## 🔧 Prerequisites

### 1. AWS Setup

- ✅ **Amazon ECR repo created** in your AWS account  
  Example: `cicd-demo` in region `us-west-2`

- ✅ **OIDC IAM role** in AWS with this trust policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::730335489670:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:rushi2828/cicd-with-github_actions-aws:*"
                }
            }
        }
    ]
}
```
![image](https://github.com/user-attachments/assets/e8a06ee4-ff2a-4f3d-b9b1-01cf4b9fd0a3)

✅ The IAM role should have permissions like:

```json
{
  "Effect": "Allow",
  "Action": [
    "ecr:GetAuthorizationToken",
    "ecr:BatchCheckLayerAvailability",
    "ecr:PutImage",
    "ecr:InitiateLayerUpload",
    "ecr:UploadLayerPart",
    "ecr:CompleteLayerUpload"
  ],
  "Resource": "*"
}
```
![image](https://github.com/user-attachments/assets/2badad5e-a424-46ac-b091-c05b6faf6356)
---
### 2. GitHub Secrets and variables
Go to:
Repo → Settings → Secrets and Variables → Actions → New repository secret

![image](https://github.com/user-attachments/assets/29d1b3d3-c2dc-4bb1-829c-26640028b6d4)

![image](https://github.com/user-attachments/assets/d0feb1a4-8e3e-4580-a04e-a8aaad6e637e)

---
### 🛠️ Example Dockerfile

```dockerfile
# Use Amazon Linux 2 Lambda base image for Python 3.12
FROM public.ecr.aws/lambda/python:3.12

# Set working directory inside the container
WORKDIR /var/task

# Copy application files

COPY lambda_function.py .

COPY requirements.txt .

# Install dependencies
RUN pip install --upgrade pip && pip install -r requirements.txt

# Set the Lambda handler
CMD ["lambda_function.lambda_handler"]
```
---
### 🧪 Create ECR Repository
![image](https://github.com/user-attachments/assets/9c383c11-f020-4a09-aced-f3fb340969f4)

---

### ⚙️ Add Github Action Workflow
Create file `.github/workflows/deploy.yml`
```
name: Deploy App 

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
          role-to-assume: arn:aws:iam::730335489670:role/EC2_Power_Role
          role-session-name: GitHubActions
          aws-region: us-west-2
          audience: sts.amazonaws.com  

      - name: Set Image Tag
        run: echo "IMAGE_TAG=$(date +%Y%m%d%H%M%S)" >> $GITHUB_ENV

      - name: Login to Amazon ECR
        run: |
          aws ecr get-login-password --region ${{ vars.AWS_REGION }} | docker login --username AWS --password-stdin ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ vars.AWS_REGION }}.amazonaws.com

      - name: Build and Push Docker Image to ECR
        run: |
          docker build -t flask-app .
          docker tag flask-app:latest ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ vars.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}
          docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ vars.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}
```
---

### 🙌 Verify ECR Image/s

![image](https://github.com/user-attachments/assets/6451cfb2-01e1-475f-8637-f8075fe23324)

---

### 📄 License
This project is licensed under the MIT License.

---
