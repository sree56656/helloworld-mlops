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

================================ Model deployment and model serving

making the trained model avaqilable to users

prob defn - data collection - data cleaning - feature eng - model selection - model training - model evaluation - hyper p tuning - deploying - observing

Deployment steps:
    1. packaging model (.pkl, containerization)
    2. pushing to registry
    3. deploying
        model serving is also a part of deployment
            1. Runtime server
            2. enough resources
            3. API available to users
            4. Ingress
            5. LB
            6. Auto scaling
            7. CDN etc.,


=============== popular ways to deploy and serve models using VMs

1. Data scientist creates model(creates scripts to create model) 
2. ML engineers create backed to this model (develop APIs using flask/fastapi)
    strategy-1 (VMs):
        ML engineers run the scripts creates artifacts and stores in VMs - /predicts - MLOPs engineers creates wsgi -> LB with in VPC
    strategy-2 (k8s): 
        k8s - containers - pod - svc - ingress
    stratrgy-3 (managed MLOps): 
        amazon sagemaker (no infra required for model deployment and serving)
    strategy-4: kserve(k8s/cloud native approach)
        it uses CRDs(custom resource defns) with this we can offload lot of k8s setups
3. model deployment

* ============= Order to create resources  

    1. VPC  
    2. subnets in vpc  
        when ALB should be associated with 2 diff subnets in diff AZ  
        to make subnets public - dest route associated with igw  
    3. IGW  
    4. Route table (create route with in route associate the destination through igw)  
        attach route tables to subnets  
    5. security group(port 22 and 80)  
        if any issue with model deployment we need to login to VM and trouble shoot issue
        attach SG to subnets  
    6. launch template  
    7. target group  
    8. ALB  
    9. ASG ( and assiciate with TG )  
    ALB -> TG -> ASG (VMs created though ASG)  

    listener rule -> ALB -> target group  

** kserve
1. It helps MLOps engineers in  
           deploying  
           serving  
           inferencing

2. as of now we have done manual activities
       flask app
       wsgi
       nginx
       deploy
       ASG
       LB

3. Kserve - CRD (.pkl file location) - it deploys in 2-3 mins
4. it works with all kind of frameworks - scikit-learn(.pkl)  
                                           tenorflow  
                                           xgboost  
                                           pytorch
5. It also supports autoscaling it supports knative/keda under the hood
