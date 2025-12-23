# Student Performance Indicator - End to End ML Project

This project is an end-to-end Machine Learning application that predicts student performance based on various parameters. It includes a complete pipeline from data ingestion to model training and deployment.

## Table of Contents

- [Project Structure](#project-structure)
- [Local Setup](#local-setup)
- [Deployment](#deployment)
  - [Method 1: AWS Elastic Beanstalk](#method-1-aws-elastic-beanstalk)
  - [Method 2: AWS ECR + EC2 with GitHub Actions](#method-2-aws-ecr--ec2-with-github-actions)
  - [Method 3: Azure Container Registry + Azure Web App](#method-3-azure-container-registry--azure-web-app)

## Project Structure

```
├── artifacts/          # Generated artifacts (data, models)
├── catboost_info/      # Catboost model training logs
├── logs/               # Application logs
├── notebook/           # Jupyter notebooks for EDA and experimentation
├── src/                # Source code
│   ├── components/     # ML components (ingestion, transformation, training)
│   ├── pipeline/       # Training and prediction pipelines
│   ├── utils.py        # Utility functions
│   ├── logger.py       # Logging configuration
│   └── exception.py    # Custom exception handling
├── templates/          # HTML templates for Flask app
├── app.py              # Flask application entry point
├── Dockerfile          # Docker configuration
├── requirements.txt    # Python dependencies
└── setup.py            # Package setup
```

## Local Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/AnupDangi/MLProjectEndtoEndDeployment.git
   cd MLProjectEndtoEndDeployment
   ```

2. **Create a virtual environment**

   ```bash
   conda create -p venv python==3.8 -y
   conda activate venv/
   ```

3. **Install dependencies**

   ```bash
   pip install -r requirements.txt
   ```

4. **Run the application**
   ```bash
   python app.py
   ```
   Access the application at `http://127.0.0.1:5000`

---

## Deployment

### Method 1: AWS Elastic Beanstalk

This method deploys the Flask application to AWS Elastic Beanstalk.

1. **Initialize EB CLI**
   ```bash
   eb init -p python-3.8 student-performance-app
   ```
2. **Create Environment**
   ```bash
   eb create student-performance-env
   ```
3. **Access Application**
   - Once deployed, the application will be accessible via the provided Elastic Beanstalk URL (Public IP/Domain).
   - Configuration is handled via `.ebextensions` (if added) or the AWS Console.

### Method 2: AWS ECR + EC2 with GitHub Actions

This method uses a Self-Hosted Runner on an EC2 instance to pull the Docker image from ECR and run it.

#### Prerequisites

1. **AWS ECR Repository**: Create a repository (e.g., `studentperformance`).
2. **IAM User**: Create a user with `AmazonEC2ContainerRegistryFullAccess` and `AmazonEC2FullAccess`. Save the Access Key and Secret Key.
3. **EC2 Instance**: Launch an Ubuntu instance and install Docker.

#### Setup Steps

1. **Configure EC2 as Self-Hosted Runner**:

   - Go to GitHub Repo > Settings > Actions > Runners > New self-hosted runner.
   - Run the provided commands on your EC2 instance to register it.

2. **GitHub Secrets**:
   Add the following secrets to your GitHub repository:

   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_REGION` (e.g., `us-east-1`)
   - `AWS_ECR_LOGIN_URI` (e.g., `123456789012.dkr.ecr.us-east-1.amazonaws.com`)
   - `ECR_REPOSITORY_NAME`

3. **Workflow**:
   The GitHub Action workflow will:

   - Build the Docker image.
   - Push the image to AWS ECR.
   - Launch the container on the EC2 instance (Self-hosted runner).

   _Note: Ensure port 5000 (or your app port) is open in the EC2 Security Group._

### Method 3: Azure Container Registry + Azure Web App

This method builds the container, pushes it to Azure Container Registry (ACR), and deploys it to an Azure Web App.

#### Prerequisites

1. **Azure Container Registry**: Create an ACR instance.
2. **Azure Web App**: Create a Web App for Containers.

#### Setup Steps

1. **Get Azure Credentials**:

   - Retrieve the ACR Login Server, Username, and Password.
   - Get the Publish Profile for the Azure Web App.

2. **GitHub Secrets**:
   Add the following secrets to your GitHub repository:

   - `AZURE_CONTAINER_REGISTRY_SERVER` (e.g., `testdockerkrish.azurecr.io`)
   - `AZURE_CONTAINER_REGISTRY_USERNAME`
   - `AZURE_CONTAINER_REGISTRY_PASSWORD`
   - `AZURE_WEBAPP_PUBLISH_PROFILE`

3. **Workflow (`azure.yaml`)**:
   The workflow is configured to:

   - Login to Azure Container Registry.
   - Build and push the Docker image.
   - Deploy the image to the Azure Web App.

   ```yaml
   # Example snippet from .github/workflows/azure.yaml
   steps:
     - name: Build and push container image to registry
       uses: docker/build-push-action@v2
       with:
         push: true
         tags: ${{ secrets.AZURE_CONTAINER_REGISTRY_SERVER }}/studentperformance:${{ github.sha }}
         file: ./Dockerfile
   ```
