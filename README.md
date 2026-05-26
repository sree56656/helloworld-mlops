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
        