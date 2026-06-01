from sklearn.datasets import load_iris
import pandas as pd
>>> from sklearn.datasets import load_iris
>>> import pandas as pd
>>> iris = load_iris()
>>> df = pd.DataFrame(iris.data, columns=iris.feature_names)
>>> df["target"] = iris.target
>>> print(df.head())
>>> print("\nTarget Names:", iris.target_names) #to print target names
=======

train.py - to build the model which creates model.pkl
run_model.py - execute/evaluate the model 
app.py - to use the model
======

**To use the model:**
curl -X POST "http://127.0.0.1:5001/predict" -H "Content-Type: application/json" -d '{"features":[10,1,5,2]}'
{"prediction":1}

================

MLOps egineers help:
    1. Data engineers - to train and saving the model
    2. ML engineers - Deploying the model

ML engineers - create APIs to integrate with backend
MLOps engineers - 
  1. containerize
  2. Iac - VM, k8s etc
  3. k8s manifests
  4. k8s - n/w, LB, security, API Gateway etc.,

================

MLOps - Course notes
https://github.com/iam-veeramalla/mlops-zero-to-hero/tree/main

================
Why DVC why not Git:
    1. Git is not good at handling large files
        usually data(csv) files are in GBs of TBs
    2. Git will be slow in pushing
    3. Git is not cost effective when dealing with large data

DVC:
    1. Stores data om cloud sources(s3, Blob, Gcs)
    2. these are designed to store large files
    3. version control is possible
    4. it is cheap
    5. highly durable

how to use:
    1. folder > dvc add (creates .dvc and dvc config file) these should be available to all the data engineers
                dvc push
                dvc pull
    2. Git has
        * python files
        * APIs
        * .dvc
        * dvc config
    3. https://github.com/iam-veeramalla/Wine-Prediction-Model


=========== MLFLOW ========
pip install mlflow
launch ui: it uses sqlite db for tracking purpose
mlflow ui --backend-store-uri sqlite:///mlflow.db --port 7006

======= mlflow installation with helm ============
installation with sqlite:
https://community-charts.github.io/docs/charts/mlflow/basic-installation

kind create cluster --name=basic-mlflow-cluster

helm repo add community-charts https://community-charts.github.io/helm-charts
helm repo update
helm install mlflow-community community-charts/mlflow

kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
mlflow-basic-install % kubectl port-forward pod/mlflow-community-f4b7b88d6-j2mjd 7006:5000 --address 0.0.0.0

====================== mlflow production setup with postgres doc (Stateful architecture) ===
steps:
    1. Setup RDS in AWS (by default it comes with postgres DB)
        create from UI
        $ psql -h <RDS host> -p 5432 -U postgres
        $ \l ==> list of databases 
    2. create mlflow DB within postgres
        $ CREATE DATABASE mlflow;
    3. create mlflow user n DB and add permissions to mlflow DB
        $ CREATE USER mlflow_user WITH PASSWORD 'your_secure_password';
        $ GRANT ALL PRIVILEGES ON DATABASE mlflow TO mlflow_user;
        $ GRANT ALL PRIVILEGES ON SCHEMA public TO mlflow_user;  

    4. install k8s cluster and run mlflow server in it - provide access to mlflow DB using mlflow user
        $ kind create cluster --name=production-mlflow-cluster
        $ kubectl create ns mlflow
        $ helm install mlflow community-charts/mlflow \
            --namespace mlflow \
            --set backendStore.databaseMigration=true \
            --set backendStore.postgres.enabled=true \
            --set backendStore.postgres.host=your-postgres-host \
            --set backendStore.postgres.port=5432 \
            --set backendStore.postgres.database=mlflow \
            --set backendStore.postgres.user=mlflow_user \
            --set backendStore.postgres.password=your_secure_password
        
        $ kubectl -n mlflow port-forward pod/mlflow-community-f4b7b88d6-j2mjd 7006:5000 --address 0.0.0.0 
        $ kubectl get pods -n mlflow -w
            1. init container = verify if the connection established with postgres
            2. mlflow container
