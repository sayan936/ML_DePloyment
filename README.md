# Network Security ML Deployment

A production-ready machine learning deployment pipeline for phishing detection using best practices from development through production deployment.

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Components](#components)
- [Setup & Installation](#setup--installation)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)
- [Running the Application](#running-the-application)
- [Deployment Pipeline](#deployment-pipeline)
- [MLflow & Experiment Tracking](#mlflow--experiment-tracking)
- [Best Practices Implemented](#best-practices-implemented)
- [Docker Deployment](#docker-deployment)
- [AWS Deployment](#aws-deployment)
- [Troubleshooting](#troubleshooting)

---

## Project Overview

This project demonstrates a complete ML deployment pipeline for **network security/phishing detection** with enterprise-grade best practices:

- ✅ Modular architecture with clear separation of concerns
- ✅ Comprehensive data validation and drift detection
- ✅ Model versioning and experiment tracking with MLflow
- ✅ FastAPI REST endpoints for predictions
- ✅ Containerized deployment with Docker
- ✅ AWS ECR/ECS deployment ready
- ✅ Error handling and logging throughout
- ✅ Environment configuration management
- ✅ MongoDB data integration

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Data Pipeline                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Data Source → Ingestion → Validation → Transformation       │
│  (MongoDB)        ↓            ↓             ↓               │
│              Feature Store  Drift Report   Preprocessor       │
│                                                              │
│                    ↓                                          │
│             ┌──────────────┐                                 │
│             │ Model Trainer │ → MLflow Tracking (Dagshub)   │
│             └──────────────┘                                 │
│                    ↓                                          │
│           ┌─────────────────┐                                │
│           │ Trained Model   │ → Model Registry               │
│           └─────────────────┘                                │
│                    ↓                                          │
│        ┌────────────────────────┐                            │
│        │    FastAPI Endpoints    │                           │
│        │  • /train (training)    │                           │
│        │  • /predict (inference) │                           │
│        │  • /docs (API docs)     │                           │
│        └────────────────────────┘                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Components

### 1. Data Ingestion (`networksecurity/components/data_ingestion.py`)
- Reads data from MongoDB
- Exports to feature store (CSV)
- Splits data into train/test sets (80/20)

### 2. Data Validation (`networksecurity/components/data_validation.py`)
- Schema validation against YAML schema
- Drift detection report generation
- Data quality checks

### 3. Data Transformation (`networksecurity/components/data_transformation.py`)
- Feature engineering and preprocessing
- Standardization/normalization
- NumPy array conversion for model training
- Preprocessor object serialization

### 4. Model Trainer (`networksecurity/components/model_trainer.py`)
- Hyperparameter tuning with GridSearchCV
- Multiple model evaluation (Random Forest, Gradient Boosting, Logistic Regression, etc.)
- MLflow experiment tracking
- Model registry integration with Dagshub

### 5. Prediction (`networksecurity/pipeline/batch_prediction.py`)
- Batch prediction capabilities
- Model and preprocessor loading

---

## Setup & Installation

### Prerequisites
- Python 3.8+
- MongoDB Atlas account (for data storage)
- Dagshub account (for MLflow tracking)
- AWS account (for ECR/ECS deployment)

### Local Development Setup

1. **Clone the repository**
```bash
git clone <repository-url>
cd ML_DePloyment
```

2. **Create virtual environment**
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**
```bash
pip install -r requirements.txt
```

4. **Configure environment variables**
Create a `.env` file in the root directory:
```env
# MongoDB Configuration
MONGO_DB_URL="mongodb+srv://<username>:<password>@cluster0.mongodb.net/?retryWrites=true&w=majority"

# MLflow Configuration
MLFLOW_TRACKING_URI="https://dagshub.com/<username>/<repo-name>.mlflow"

# AWS Configuration (for deployment)
AWS_ACCESS_KEY_ID=<your-access-key>
AWS_SECRET_ACCESS_KEY=<your-secret-key>
AWS_REGION=us-east-1
AWS_ECR_LOGIN_URI=<your-ecr-uri>
ECR_REPOSITORY_NAME=networkssecurity
```

**Note:** The `.env` file is git-ignored for security. Never commit credentials.

---

## Project Structure

```
ML_DePloyment/
├── networksecurity/
│   ├── __init__.py
│   ├── components/
│   │   ├── data_ingestion.py
│   │   ├── data_transformation.py
│   │   ├── data_validation.py
│   │   └── model_trainer.py
│   ├── constant/
│   │   └── training_pipeline/
│   │       └── __init__.py
│   ├── entity/
│   │   ├── artifact_entity.py       # Output artifacts from each component
│   │   └── config_entity.py         # Configuration classes
│   ├── exception/
│   │   └── exception.py             # Custom exception handling
│   ├── logging/
│   │   └── logger.py                # Logging configuration
│   ├── pipeline/
│   │   ├── training_pipeline.py     # Orchestrates the training flow
│   │   └── batch_prediction.py      # Batch prediction pipeline
│   ├── utils/
│   │   ├── main_utils/
│   │   │   └── utils.py             # General utilities
│   │   └── ml_utils/
│   │       ├── metric/
│   │       │   └── classification_metric.py
│   │       └── model/
│   │           └── estimator.py     # Model wrapper
│   └── cloud/
│       └── s3_syncer.py             # AWS S3 integration
├── data_schema/
│   └── schema.yaml                  # Data validation schema
├── Network_Data/
│   └── phisingData.csv              # Sample dataset
├── templates/
│   └── table.html                   # HTML template for predictions
├── Artifacts/                       # Generated artifacts (git-ignored)
│   └── {timestamp}/
│       ├── data_ingestion/
│       ├── data_validation/
│       ├── data_transformation/
│       └── model_trainer/
├── final_model/                     # Final trained model (git-ignored)
├── logs/                            # Application logs (git-ignored)
├── app.py                           # FastAPI application
├── main.py                          # Training pipeline entry point
├── Dockerfile                       # Container configuration
├── requirements.txt                 # Python dependencies
├── .env                             # Environment variables (git-ignored)
├── .gitignore                       # Git ignore rules
└── README.md                        # This file
```

---

## Development Workflow

### 1. **Data Preparation**
```bash
python push_data.py  # Push phishing data to MongoDB
```

### 2. **Run Training Pipeline**
```bash
python main.py
```

This triggers:
- Data ingestion from MongoDB
- Schema validation
- Drift detection
- Data transformation
- Model training with hyperparameter tuning
- Experiment logging to MLflow/Dagshub

### 3. **Monitor Experiments**
View all experiments at: `https://dagshub.com/<username>/<repo-name>.mlflow`

### 4. **Run FastAPI Application**
```bash
python app.py
# Or with uvicorn explicitly
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

Access API documentation: `http://localhost:8000/docs`

---

## Running the Application

### Training Endpoint
```bash
curl -X GET http://localhost:8000/train
```

Response:
```json
"Training is successful"
```

### Prediction Endpoint
```bash
curl -X POST http://localhost:8000/predict \
  -F "file=@test_data.csv"
```

Returns an HTML table with predictions.

### API Documentation
Navigate to: `http://localhost:8000/docs`

---

## Deployment Pipeline

### Stage 1: Development
- ✅ Local testing with virtual environment
- ✅ Git version control with `.gitignore` for sensitive files
- ✅ Error handling and logging

### Stage 2: Experimentation
- ✅ MLflow tracking for model experiments
- ✅ Dagshub integration for remote experiment storage
- ✅ Hyperparameter tuning and model comparison

### Stage 3: Containerization
- ✅ Docker image creation
- ✅ Container registry (AWS ECR)

### Stage 4: Orchestration
- ✅ AWS ECS deployment
- ✅ Environment configuration management

---

## MLflow & Experiment Tracking

### Tracked Metrics
- F1 Score
- Precision
- Recall Score

### Model Registry
Models are automatically registered in Dagshub with:
- Model artifacts (sklearn pickle files)
- Training metrics
- Model versions

### Accessing Results
```
https://dagshub.com/<username>/<repo-name>.mlflow/#/experiments/0
```

---

## Best Practices Implemented

### 🎯 Code Organization
- Modular component-based architecture
- Clear separation of concerns (data, model, pipeline, utils)
- Reusable entity classes for configuration and artifacts

### 🔒 Error Handling & Logging
- Custom `NetworkSecurityException` for consistent error handling
- Dedicated logging module with timestamped log files
- Try-except blocks throughout pipeline
- Exception chains preserved for debugging

### 📊 Data Management
- Timestamped artifact directories for reproducibility
- Separate train/test file paths
- Feature store for preprocessed data
- Schema-based data validation

### 🧪 Model Development
- Multiple model comparison framework
- Hyperparameter tuning with GridSearchCV
- Train/test split with consistent evaluation
- Metrics calculation for multiple folds

### 🚀 Experiment Tracking
- MLflow integration for automatic logging
- Remote model registry (Dagshub)
- Metric persistence and comparison
- Model versioning support

### 🐳 Containerization
- Dockerfile for consistent deployment
- Environment variable configuration
- Multi-stage build optimization

### ☁️ Cloud Deployment
- AWS ECR integration for image registry
- Docker authentication setup
- Container push automation

### 🔐 Security
- Environment variable management (.env file)
- Git ignore for credentials and generated files
- MongoDB connection string escaping for special characters
- No hardcoded secrets in code

### 📝 Configuration Management
- Config entity classes for pipeline configuration
- YAML schema files for validation
- Centralized constant definitions
- Environment-based configuration

### 🔄 Reproducibility
- Fixed random seeds (where applicable)
- Timestamped artifacts for tracking
- Complete pipeline logging
- Model and preprocessor serialization

---

## Docker Deployment

### Build Docker Image
```bash
docker build -t networkssecurity:latest .
```

### Run Container Locally
```bash
docker run -p 8000:8000 \
  --env-file .env \
  networkssecurity:latest
```

### Push to AWS ECR
```bash
# Login to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <ECR_URI>

# Tag image
docker tag networkssecurity:latest <ECR_URI>/networkssecurity:latest

# Push to registry
docker push <ECR_URI>/networkssecurity:latest
```

---

## AWS Deployment

### Prerequisites
1. AWS ECR repository created
2. AWS credentials configured
3. ECS cluster and task definition setup

### Deployment Steps
1. Push Docker image to ECR (see Docker section)
2. Update ECS task definition with new image URI
3. Deploy new task to ECS cluster
4. Monitor logs in CloudWatch

### Environment Variables
Set in ECS task definition:
```
MONGO_DB_URL=<connection-string>
MLFLOW_TRACKING_URI=<dagshub-uri>
```

---

## Troubleshooting

### MongoDB Connection Error
**Error:** `InvalidURI: Username and password must be escaped according to RFC 3986`

**Solution:** URL-encode special characters in password:
```python
from urllib.parse import quote_plus
password = "Sayanchak@97@"
encoded = quote_plus(password)
# Use in connection string with @ escaped as %40
```

### MLflow Registration Error
**Error:** `TypeError: bad argument type for built-in operation`

**Solution:** Ensure `registered_model_name` is a string, not a model object:
```python
mlflow.sklearn.log_model(best_model, "model", 
                         registered_model_name=str(model_name))
```

### Missing Dependencies
```bash
pip install -r requirements.txt --upgrade
```

### Permission Denied on Docker
```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## Key Files & Their Purpose

| File | Purpose |
|------|---------|
| `main.py` | Entry point for training pipeline |
| `app.py` | FastAPI application with REST endpoints |
| `push_data.py` | Script to push data to MongoDB |
| `Dockerfile` | Container configuration |
| `requirements.txt` | Python dependencies |
| `.env` | Environment variables (git-ignored) |
| `.gitignore` | Git ignore rules |

---

## Future Enhancements

- [ ] Add model validation metrics to API
- [ ] Implement automated retraining on schedule
- [ ] Add database connection pooling
- [ ] Implement request/response caching
- [ ] Add comprehensive API tests
- [ ] Setup CI/CD pipeline with GitHub Actions
- [ ] Add monitoring and alerting
- [ ] Implement A/B testing framework

---

## Contributing

1. Create a feature branch: `git checkout -b feature/your-feature`
2. Commit changes: `git commit -am 'Add feature'`
3. Push to branch: `git push origin feature/your-feature`
4. Submit a pull request

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## Support

For issues or questions:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review logs in the `logs/` directory
3. Check MLflow experiments at Dagshub
4. Open an issue in the repository

---

## Author

Sayan Chakraborty  
ML Deployment Project - Network Security Phishing Detection

---

**Last Updated:** August 2026  
**Status:** Production Ready
