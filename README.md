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