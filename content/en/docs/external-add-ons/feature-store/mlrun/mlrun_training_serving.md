+++
title = "Training & Serving" 
description = "Use MLRun to train and serve your ML models"
weight = 15

+++


When working on a new model, you need to ensure the experiment’s reproducibility as well as the ability to recreate 
the feature and and the model-environment. MLRun’s feature store enables all that in a simple and automated fashion.

After defining the [feature sets](https://docs.mlrun.org/en/latest/feature-store/transformations.html) and 
proposing a [feature vector](https://docs.mlrun.org/en/latest/feature-store/feature-vectors.html) for the experiment, 
use the feature store to automatically extract a versioned **offline** static dataset, based on the parquet target 
that is defined in the feature sets for training.

For serving, after you validate the feature vector, use the **online** feature service based on the **nosql** target 
defined in the feature set for real-time serving.

Using this feature-store-centric process and one computation graph definition for a feature set, you receive an 
automatic online and offline implementation for the feature vectors. 

These feature vectors hold data that is versioned both in terms of the actual graph that was used to calculate each 
data point, and the offline datasets that were created to train each model.

## The solution's flow chart 

<img src="/docs/external-add-ons/feature-store/mlrun/images/feature-store-training-v2.png"
  alt="Training & Serving Solution"
  class="mt-3 mb-3 border border-info rounded">

## Creating an offline dataset

An “offline” dataset is a specific instance of your feature vector definition. To create this instance, use the 
feature store’s `get_offline_features(<feature_vector>, <target>)` function on your feature vector using the 
`store://<project_name>/<feature_vector>` reference and an offline target (as in Parquet, CSV, etc.).

```shell
import mlrun.feature_store as fstore
 
feature_vector = '<feature_vector_name>'
offline_fv = fstore.get_offline_features(feature_vector=feature_vector, target=ParquetTarget())
```

Behind the scenes, `get_offline_features()` runs a local or Kubernetes job (specified by the `run_configparameter`) 
to retrieve all the relevant data from the feature sets, merge it, and return it to the specified `target` . Targets 
can be a local parquet, AZ Blob store or any other type of available storage.

Once instantiated with a target, the feature vector holds a reference to the instantiated dataset and references it as 
its current offline source.

You can also use MLRun’s `log_dataset()` to log the specific dataset to the project as a specific dataset resource.

## Training 

Training your model using the feature store is a fairly simple task. 

To retrieve a feature vector’s offline dataset, use MLRun’s DataItem mechanism, referencing the feature vector and 
specifying the dataset as a DataFrame.

```shell
df = mlrun.get_dataitem(f'store://feature-vectors/{project}/patient-deterioration').as_df()
```

When retrieving the dataset in your training function, put the feature vector reference as an input to the function and 
use the `as_df()` function to retrieve it automatically.

```shell
# A sample MLRun training function
def my_training_function(context, # MLRun context
                         dataset, # your feature vector reference
                         **kwargs):
    
    # retrieve the dataset
    df = dataset.as_df()
 
    # The rest of your training code...
```

Now you can create your MLRun function and run it locally, or over the Kubernetes cluster, as follows: 

```shell
# Creating the training MLRun function with your code
fn = mlrun.code_to_function('training', 
                            kind='job',
                            handler='my_training_function')
 
# Creating the task to run your function with your dataset
task = mlrun.new_task('training', 
                      inputs={'dataset': f'store://feature-vectors/{project}/{feature_vector_name}'}) # The feature 
                      # vector is given as an input to the function
 
# Running the function over the kubernetes cluster
fn.run(task) # Set local=True to run locally
```

## Get online features 

The online features are created ad-hoc using MLRun’s feature store online feature service, and are served from the 
**nosql** target for real-time performance needs.

First create an online feature service with your feature vector.

```shell
import mlrun.feature_store as fstore
 
svc = fstore.get_online_feature_service(<feature vector name>)
```

Then use the feature vector entity to get the latest feature vector for it. You can pass a 
list of `{<key name>:<key value>}` pairs to receive a batch of feature vectors.

```shell
fv = svc.get([{<key name>: <key value>}])
```

## Incorporating to serving model 

With MLRun you can easily serve your models using the 
[model server example](https://github.com/mlrun/functions/blob/master/v2_model_server/v2_model_server.ipynb). 
Define a serving model class and the computational graph required to run your entire prediction pipeline, 
and deploy it as serverless functions using [Nuclio](https://github.com/nuclio/nuclio).

To embed the online feature service in your model server, just create the feature vector service once when the model 
initializes, and then use it to retrieve the feature vectors of incoming keys.

You can import ready-made classes and functions from the MLRun [function marketplace](https://github.com/mlrun/functions) 
 or you can write your own. Here's an example of a scikit-learn based model server (taken from the 
[feature store demo](https://docs.mlrun.org/en/latest/feature-store/training-serving.html#incorporating-to-serving-model)):

```shell
from cloudpickle import load
import numpy as np
import mlrun
import os
 
class ClassifierModel(mlrun.serving.V2ModelServer):
    
    def load(self):
        """load and initialize the model and/or other elements"""
        model_file, extra_data = self.get_model('.pkl')
        self.model = load(open(model_file, 'rb'))
        
        # Setup FS Online service
        self.feature_service = mlrun.feature_store.get_online_feature_service('patient-deterioration')
        
        # Get feature vector statistics for imputing
        self.feature_stats = self.feature_service.vector.get_stats_table()
        
    def preprocess(self, body: dict, op) -> list:
        # Get patient feature vector 
        # from the patient_id given in the request
        vectors = self.feature_service.get([{'patient_id': patient_id} for patient_id in body['inputs']])
        
        # Impute inf's in the data to the feature's mean value
        # using the collected statistics from the Feature store
        feature_vectors = []
        for fv in vectors:
            new_vec = []
            for f, v in fv.items():
                if np.isinf(v):
                    new_vec.append(self.feature_stats.loc[f, 'mean'])
                else:
                    new_vec.append(v)
            feature_vectors.append(new_vec)
            
        # Set the final feature vector as your inputs
        # to pass to the predict function
        body['inputs'] = feature_vectors
        return body
 
    def predict(self, body: dict) -> list:
        """Generate model predictions from sample"""
        feats = np.asarray(body['inputs'])
        result: np.ndarray = self.model.predict(feats)
        return result.tolist()
```

Which you can deploy with:

```shell
# Create the serving function from your code above
fn = mlrun.code_to_function(<function_name>, 
                            kind='serving')
 
# Add a specific model to the serving function
fn.add_model(<model_name>, 
             class_name='ClassifierModel',
             model_path=<store_model_file_reference>)
 
# Enable MLRun's model monitoring
fn.set_tracking()
 
# Add the system mount to the function so
# it will have access to your model files
fn.apply(mlrun.mount_v3io())
 
# Deploy the function to the cluster
fn.deploy()
```

And test using:

```shell
fn.invoke('/v2/models/infer', body={<key name>: <key value>})
```

