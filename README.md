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
