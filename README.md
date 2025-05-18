**Vehicle Prediction & Deployment Pipeline**
*A comprehensive end-to-end solution covering environment setup, data ingestion, processing, model training, and CI/CD deployment.*

---

## ▶️ Project Overview

This repository encapsulates a modular machine learning workflow for vehicle-related data, extending from initial setup through production deployment on AWS. The architecture ensures maintainability, scalability, and clear separation of concerns:

1. **Environment & Package Configuration**
2. **MongoDB Data Ingestion**
3. **Logging & Exception Management**
4. **Data Ingestion Component**
5. **Data Validation & Transformation**
6. **Model Training & Evaluation**
7. **AWS Integration (S3, EC2, IAM, ECR)**
8. **CI/CD with GitHub Actions**
9. **Prediction Service & Deployment**

---

## 📂 Repository Structure

```
vehicle-pipeline/
│
├── .github/
│   └── workflows/
│       └── aws.yaml            # CI/CD workflow definition
│
├── notebooks/
│   ├── mongoDB_demo.ipynb      # Demonstrates pushing data to MongoDB
│   ├── eda_feature_engg.ipynb  # Exploratory Data Analysis & Feature Engineering
│   └── …                       # Additional Jupyter notebooks
│
├── src/
│   ├── components/             # Core pipeline components
│   │   ├── data_ingestion.py   # Fetches data from MongoDB into DataFrame
│   │   ├── data_validation.py  # Validates raw data against schema
│   │   ├── data_transformation.py
│   │   ├── model_trainer.py
│   │   ├── model_evaluation.py
│   │   └── model_pusher.py      # Uploads trained model to S3
│   │
│   ├── configuration/          # Configuration utilities
│   │   ├── constants/          # Project-wide constants (DB URIs, thresholds, AWS settings)
│   │   ├── aws_connection.py   # AWS S3 connection logic
│   │   └── mongo_db_connections.py  # MongoDB connection helper
│   │
│   ├── data_access/            # Abstracts data retrieval
│   │   └── proj1_data.py       # Retrieves and transforms MongoDB documents into DataFrame
│   │
│   ├── entities/               # Dataclasses defining configuration and artifacts
│   │   ├── config_entity.py    # Config‐level classes (e.g. DataIngestionConfig)
│   │   ├── artifact_entity.py  # Artifact‐level classes (e.g. DataIngestionArtifact)
│   │   └── s3_estimator.py      # S3 push/pull utilities
│   │
│   ├── exceptions/             # Custom exception classes
│   │   └── custom_exception.py
│   │
│   ├── logger/                 # Centralized logger configuration
│   │   └── logging_config.py
│   │
│   ├── pipeline/               # Orchestrates end-to-end workflow
│   │   └── training_pipeline.py
│   │
│   ├── utils/                  # Helper functions (schema validation, I/O, etc.)
│   │   └── main_utils.py
│   │
│   └── app.py                  # Main entry point to run ingestion & training
│
├── requirements.txt            # Required Python packages
├── setup.py & pyproject.toml   # Local package setup and metadata
├── Dockerfile                  # Containerization definition
├── .dockerignore               # Files & folders to ignore in Docker build
└── README.md                   # This documentation
```

---

## 🛠 Environment Setup

1. **Project Template**

   * Execute `template.py` to scaffold the project structure.
   * This creates all necessary directories and placeholder files.

2. **Local Package Configuration**

   * Populate `setup.py` and `pyproject.toml` for local imports and package metadata.
   * Reference `crashcourse.txt` for guidance on both files.

3. **Conda Environment Creation**

   * Create a dedicated environment named `vehicle` (Python 3.10).
   * Activate the environment before installing dependencies.

4. **Dependency Installation**

   * Add all required third-party modules to `requirements.txt`.
   * Install them using your package manager to ensure reproducible installs.
   * Verify successful installation by listing installed packages and confirming presence of local modules.

---

## 🗄 MongoDB Atlas Setup

1. **Account & Project Creation**

   * Sign up for MongoDB Atlas, create a new project, and proceed to build a free (M0) cluster.

2. **Cluster Deployment**

   * Choose the M0 free tier, keep default settings, and deploy.
   * Once active, configure a database user with a secure username/password.

3. **Network Access**

   * Under “Network Access,” add `0.0.0.0/0` to allow unrestricted IP access.
   * Note: For production, restrict to specific IP ranges.

4. **Connection URI**

   * Retrieve the driver connection string (Python version 3.6+) from “Connect → Drivers.”
   * Replace the placeholder with your password and save the URI.

5. **Demo Notebook**

   * In `notebooks/mongoDB_demo.ipynb`, connect to MongoDB using the URI.
   * Load a sample dataset into a pandas DataFrame, then insert it into a MongoDB collection.
   * Verify ingestion via Atlas’s “Browse Collections” view.

---

## 🔍 Logging & Exception Management

1. **Logger Configuration**

   * The `logging_config.py` module sets up a standardized logger (console + file) with timestamps and log levels.
   * Use this logger in any component to produce consistent, formatted log messages.

2. **Custom Exceptions**

   * Define `CustomException` (in `custom_exception.py`) to wrap lower-level errors, adding context and full stack traces.
   * Use try/except blocks around database connections, file I/O, and critical operations; raise `CustomException` on failure.

3. **Demo Validation**

   * Use notebooks under `notebooks/` to explore raw data, define schemas (in `config/schema.yaml`), and identify validation rules.
   * These rules inform the Data Validation component.

---

## 📥 Data Ingestion

1. **Configuration & Constants**

   * Define MongoDB connection URI and database/collection names within `constants/__init__.py`.

2. **MongoDB Connection Utility**

   * Implement connection logic in `configuration/mongo_db_connections.py`.
   * Provide a function to return an active client or raise a custom exception on failure.

3. **Data Access Module**

   * In `data_access/proj1_data.py`, fetch documents from MongoDB, convert to pandas DataFrame, and perform initial transformations.

4. **Entity Definitions**

   * In `entities/config_entity.py`, create `DataIngestionConfig` dataclass containing source collection, target paths, and relevant metadata.
   * In `entities/artifact_entity.py`, define `DataIngestionArtifact` to hold the DataFrame location, record counts, and ingestion timestamps.

5. **Ingestion Component**

   * In `components/data_ingestion.py`, use the MongoDB connection and data access code to read data, perform any lightweight cleansing, and write to a local file or in-memory artifact.
   * Return a `DataIngestionArtifact` instance capturing details of the operation.

6. **Pipeline Orchestration**

   * Add logic in `pipeline/training_pipeline.py` to instantiate `DataIngestionConfig`, invoke the ingestion component, and pass the artifact downstream.

7. **Environment Variable for URI**

   * Set the environment variable `MONGODB_URL` to your Atlas URI on your local machine (via shell or system settings).
   * This decouples secrets from source code.

---

## 📊 Data Validation & Transformation

1. **Schema Definition**

   * Populate `config/schema.yaml` with field names, data types, required columns, and value constraints.

2. **Validation Utility**

   * Implement helper functions in `utils/main_utils.py` to load the schema, verify required columns exist, check for nulls or invalid values, and generate detailed reports.

3. **Validation Component**

   * In `components/data_validation.py`, read the raw ingestion artifact, validate against the schema rules, and raise exceptions or log warnings if discrepancies are found.
   * Output a `DataValidationArtifact` summarizing pass/fail counts and any corrective actions needed.

4. **Transformation Component**

   * In `components/data_transformation.py`, apply feature engineering steps (e.g., encoding, normalization, feature selection) guided by insights from `notebooks/eda_feature_engg.ipynb`.
   * Define `DataTransformationConfig` and `DataTransformationArtifact` in `entities/`.
   * Ensure transformed data is saved to a local directory for model consumption.

---

## 🤖 Model Trainer & Evaluator

1. **Model Trainer Component**

   * In `components/model_trainer.py`, load transformed data, split into training/validation sets, initialize the chosen algorithm (e.g., RandomForest, XGBoost), and train.
   * Capture model metadata—hyperparameters, training metrics—in `ModelTrainerArtifact`.

2. **Model Evaluation Component**

   * In `components/model_evaluation.py`, compare trained model performance on a holdout set.
   * Use thresholds defined in `constants/` (e.g., minimum improvement over baseline) to decide if the model is accepted.
   * Store evaluation metrics—accuracy, precision, recall, F1—in `ModelEvaluationArtifact`.

3. **Model Pusher Component**

   * In `components/model_pusher.py`, package the trained model artifact and push it to an S3 bucket.
   * Leverage the AWS connection logic in `configuration/aws_connection.py`.
   * S3 bucket name, object key prefix, and evaluation threshold configurations reside in `constants`.

4. **Entity Definitions**

   * Create `ModelTrainerConfig`, `ModelTrainerArtifact`, `ModelEvaluationConfig`, `ModelEvaluationArtifact`, and `ModelPusherConfig` in `entities/estimator.py` (or separate files as needed).

5. **Pipeline Integration**

   * Extend `pipeline/training_pipeline.py` to sequence: ingestion → validation → transformation → training → evaluation → pushing.
   * Log each step’s start/end times and any artifacts produced.

---

## ☁️ AWS Services Configuration

1. **IAM User & Access Keys**

   * In AWS Console (region **us-east-1**), navigate to IAM and create a user named `firstproj` with `AdministratorAccess`.
   * Generate an access key/secret key pair and download the credentials CSV.
   * Expose these credentials via environment variables:

     * `AWS_ACCESS_KEY_ID`
     * `AWS_SECRET_ACCESS_KEY`

2. **Constants Update**

   * In `constants/__init__.py`, set:

     * `MODEL_EVALUATION_CHANGED_THRESHOLD_SCORE = 0.02`
     * `MODEL_BUCKET_NAME = "my-model-mlopsproj"`
     * `MODEL_PUSHER_S3_KEY = "model-registry"`

3. **S3 Bucket Creation**

   * In AWS S3 (us-east-1), create a bucket named `my-model-mlopsproj`.
   * Disable “Block all public access” if required (for public assets).
   * Use this bucket for storing serialized model objects.

4. **S3 Connection Utility**

   * In `configuration/aws_connection.py`, implement functions to upload and download objects from the specified S3 bucket using the AWS SDK (`boto3`).

5. **EC2 Instance for Deployment**

   * Launch an Ubuntu Server 24.04 instance called `vehicledata-machine` with instance type `t2.medium`.
   * Create a key pair named `proj1key`, open ports for HTTP/HTTPS, and attach a 30 GB volume.
   * SSH into the instance to install Docker and run the containerized prediction service.

---

## 🔄 CI/CD Process

1. **Docker Configuration**

   * **Dockerfile**: Defines the container image (base Python, environment setup, dependencies, and the application).
   * **`.dockerignore`**: Specifies files and directories to exclude from the Docker build context.

2. **GitHub Actions Workflow**

   * Located at `.github/workflows/aws.yaml`.
   * Triggered on every push to `main` (or specify branch filters).
   * Steps include:

     * Checking out the repository.
     * Building the Docker image.
     * Logging in to ECR.
     * Pushing the image to the ECR repository.
     * Deploying or updating the service on the EC2 instance.

3. **ECR Repository**

   * Create an ECR repository named `vehicleproj` in `us-east-1`.
   * Copy its URI for use in GitHub Actions secrets.

4. **EC2 Self-Hosted Runner**

   * On GitHub: Under **Settings → Actions → Runners**, create a new self-hosted Linux runner.
   * Copy the registration commands and execute them on the EC2 instance, assigning the runner to your repository.
   * Confirm the runner’s status in GitHub (it should show as “idle”).

5. **GitHub Secrets**

   * Add the following repository-level secrets under **Settings → Secrets → Actions**:

     * `AWS_ACCESS_KEY_ID`
     * `AWS_SECRET_ACCESS_KEY`
     * `AWS_DEFAULT_REGION`
     * `ECR_REPO` (ECR repository URI)

6. **Port Configuration**

   * In the EC2 Security Group, add an inbound rule for TCP port `5080` with source `0.0.0.0/0`.
   * This exposes the prediction service to the public internet on port 5080.

7. **CI/CD Trigger**

   * On every push to `main`, the workflow builds, pushes to ECR, and updates/prepares the EC2 container deployment.
   * The public IP of the EC2 instance plus `:5080` reveals the live prediction endpoint.

---

## 📈 Prediction Service & Usage

1. **Prediction Pipeline Structure**

   * Under `src/pipeline`, create `prediction_pipeline.py` which defines how incoming data is preprocessed, fed to the model, and postprocessed into a response.

2. **Flask/FastAPI Application**

   * In `app.py`, configure the web server:

     * Load the latest model from S3 or local path.
     * Define API endpoints (e.g., `/predict`, `/health`).
     * Include static files (HTML/CSS/JS) under `static/` and Jinja templates under `templates/`.

3. **Container Entry Point**

   * Ensure the Dockerfile launches the Flask/FastAPI app on port `5080`.
   * Verify environment variables inside the container for AWS credentials, MongoDB URI, and other secrets.

4. **Model Training Endpoint (Optional)**

   * Expose an additional route (e.g., `/train`) to trigger model retraining on-demand.
   * This endpoint can call the training pipeline, save artifacts to S3, and update the production model.

5. **Accessing the Service**

   * Once deployed, navigate to `http://<EC2_PUBLIC_IP>:5080/` to view the application homepage or prediction form.
   * Use the `/predict` endpoint (via HTTP POST) to send JSON input and receive model predictions.

---

## ✅ Quick Start Checklist

1. **Clone the Repository**

   * Navigate into the directory.

2. **Environment Setup**

   * Create & activate the Conda environment (`vehicle`).
   * Install packages from `requirements.txt`.

3. **MongoDB**

   * Configure `MONGODB_URL` environment variable.
   * Run the demo notebook to verify data ingestion.

4. **AWS Credentials**

   * Set `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in your shell.
   * Confirm you can list S3 buckets and connect to ECR.

5. **Docker & Container**

   * Build the Docker image locally.
   * Run the container and confirm the app responds on port 5080.

6. **GitHub Actions**

   * Ensure `.github/workflows/aws.yaml` is in the `main` branch.
   * Verify that the self-hosted runner is “idle” in GitHub settings.
   * Commit & push; check the Actions tab for build logs.

7. **Deployment Verification**

   * Confirm the ECR repository contains the latest image.
   * SSH into EC2, pull the new image, and restart the container.
   * Open `http://<EC2_PUBLIC_IP>:5080/` in a browser to confirm the live app.

---

## 🎯 Summary

This repository delivers:

* A reproducible environment setup with Conda and clear instructions for dependency management.
* A robust data ingestion pipeline from MongoDB, including configuration, validation, and transformation layers.
* End-to-end model development: training, evaluation, and automated pushing to S3.
* A fully containerized prediction service, deployed on an EC2 instance.
* A CI/CD pipeline with GitHub Actions and ECR, enabling seamless image builds and deployment.
* Comprehensive logging, exception handling, and modular code organization for maintainability.
