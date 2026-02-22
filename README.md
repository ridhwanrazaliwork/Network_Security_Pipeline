# Network Security Anomaly Detection Pipeline

## Overview

- An automated machine learning pipeline for detecting anomalies/intrusions in network traffic data. The system ingests network logs, validates and transforms features, trains classification models, and deploys accepted models to production.

## Architecture

- A sequential, config-driven pipeline with six core components:

- Data Ingestion → Data Validation → Data Transformation → Model Training → Model Evaluation → Model Pusher


Each component:
- Reads configuration (YAML/JSON)
- Consumes artifacts from the previous stage
- Produces artifacts for the next stage
- Validates inputs/outputs before proceeding

## Pipeline Components

| Component | Responsibilities |
|-----------|------------------|
| **Data Ingestion** | Pulls raw network data from MongoDB Atlas. Splits into train/test sets. |
| **Data Validation** | Validates schema compliance, checks for missing columns, detects data drift between train/test sets. |
| **Data Transformation** | Applies preprocessing pipeline: imputation → robust scaling → SMOTE-Tomek (for class imbalance) → saves as NumPy arrays. |
| **Model Trainer** | Trains multiple models using config-defined parameters; selects best performer based on metrics. |
| **Model Evaluation** | Compares best model against baseline accuracy threshold; accepts/rejects for deployment. |
| **Model Pusher** | Saves accepted models (`model.pkl`) and preprocessing artifacts (`preprocessing.pkl`) to production store. |

## Data Flow

```mermaid
flowchart TD
    A[Raw Network Data<br>MongoDB Atlas] --> B(Data Ingestion)
    B --> C{Data Validation<br>Schema + Drift Check}
    C -->|Valid| D[Data Transformation<br>Impute → Scale → SMOTE]
    C -->|Invalid| E[Quarantine]
    D --> F[Model Training]
    F --> G{Model Evaluation<br>vs. Baseline Accuracy}
    G -->|Accepted| H[Model Pusher → Production]
    G -->|Rejected| I[Archive/Retrain]

```

## Dataset
Dataset can be downloaded from
https://archive.ics.uci.edu/dataset/327/phishing+websites

## Key Features
- Config-driven: All stages controlled via YAML configs (paths, thresholds, model params)
- Artifact tracking: Each component outputs versioned artifacts (CSVs, .npy, .pkl)
- Drift detection: Monitors feature distribution shifts between train/test sets
- Imbalance handling: SMOTE-Tomek resampling for skewed network attack classes
- Deployment-ready: Final artifacts (model.pkl + preprocessing.pkl) packaged for AWS deployment (Docker/Kubernetes/ECS)

## Deployment Target
Models deployed to AWS infrastructure:
Docker containerization
EC2
AWS S3 for artifact storage

## 🚀 Deployment Pipeline Flow

```mermaid
graph TD
    LocalDev["💻 Local Development<br/>Desktop"]
    LocalDev -->|git push| GitHub["🐙 GitHub<br/>Repository"]
    
    GitHub -->|Webhook Trigger| Actions["⚙️ GitHub Actions<br/>CI/CD Pipeline"]
    
    Actions -->|Build| Docker["🐳 Build Docker Image<br/>FROM: Dockerfile"]
    
    Docker -->|Push| ECR["📦 AWS ECR<br/>Elastic Container Registry"]
    
    ECR -->|Pull Image| AppRunner["🚀 AWS App Runner<br/>Managed Service"]
    
    AppRunner -->|Deploy| EC2["☁️ AWS EC2<br/>Production Instance"]
    
    EC2 -->|Running| Prod["✅ Network Security<br/>Anomaly Detector Live"]
    
    style LocalDev fill:#FFE4B5,color:#000
    style GitHub fill:#FFD700,color:#000
    style Actions fill:#87CEEB,color:#000
    style Docker fill:#ADD8E6,color:#000
    style ECR fill:#FFA07A,color:#000
    style AppRunner fill:#98FB98,color:#000
    style EC2 fill:#DDA0DD,color:#000
    style Prod fill:#90EE90,color:#000
```

### Pipeline Stages Explained

1. **Local Development** → Code changes pushed to GitHub
2. **GitHub Actions** → Automated CI/CD pipeline triggered on push
3. **Docker Build** → Dockerfile builds container image
4. **ECR Push** → Docker image pushed to AWS Elastic Container Registry
5. **App Runner** → AWS App Runner pulls image and manages deployment
6. **EC2 Deployment** → Application runs on EC2 instance in production
7. **Live Service** → Network security model actively detecting anomalies

---

## 🧹 Cleanup (Important: Avoid AWS Costs)

⚠️ **To prevent unexpected charges, clean up these AWS resources after testing:**

1. **Delete AWS App Runner Service**
   - AWS Console → App Runner → Services → Select service → Delete

2. **Delete ECR Repository**
   - AWS Console → ECR → Repositories → Select repository → Delete repository

3. **Terminate EC2 Instance** (if applicable)
   - AWS Console → EC2 → Instances → Select instance → Terminate instances

4. **Remove Security Groups**
   - AWS Console → EC2 → Security Groups → Delete any custom groups

5. **Delete CloudWatch Logs**
   - AWS Console → CloudWatch → Logs → Select log group → Delete log group

6. **Disable MongoDB Atlas Cluster** (if using cloud database)
   - MongoDB Atlas → Clusters → Pause/Terminate cluster


---

## Reference
Sample project from MLOps Bootcamp course by Krish Naik