+++
title = "MLRun Feature Store" 
description = "Serverless real-time + offline feature store with scalable and high-performance data transformation layer"
weight = 10

+++


The common flow to work with MLRun’s feature store is: 
1. Define the features, transformations, and validation logic
2. Ingest data (Dataframe, Kubernetes, Nuclio)
3. Create feature vectors for training
4. Serve models

## Define Features, Transformations, and Validation Logic

The common flow when working with the feature store is to first define the feature set with its source, transformation 
graph, and targets. To test the execution process, call the `infer` method with a sample DataFrame. This runs all 
operations in memory without storing the results. Once The graph is defined, it’s time to ingest the data.

## Ingest Data
There are a few methods of ingesting data.

You can ingest data directly from a DataFrame by calling the feature set `ingest` method. 

You can define an ingestion process that runs as a Kubernetes job. This is useful if there is a large ingestion process, 
or if there is a recurrent ingestion, and you want to schedule the job.

MLRun can also leverage [Nuclio](https://nuclio.io/) to perform real-time ingestion by calling the 
`deploy_ingestion_service function`. During the serving you can update feature values (e.g. to update 
a sliding window aggregation as part of a model serving process), and not just read them. 

## Create a Feature Vector, and Train Models

Define the feature vector by calling the `get_offline_features` function, enabling the joining of features across 
different feature sets. 

Train a model with the feature vector data, by providing the input using the format 
`'store://feature-vectors/{project}/{feature_vector_name}'` to train a model with the feature vector data.

## Serve Models

Define a serving class derived from `mlrun.serving.V2ModelServer`.

In the class `load` method, call the `get_online_feature_service` function with the vector name. This returns a feature service object. 

In the class `preprocess` method, call the feature service `get` method to get the values of those features.


## Further MLRun  Documentation 

* [Feature Store](https://docs.mlrun.org/en/latest/feature-store/feature-store.html)

* [Training & Serving](https://docs.mlrun.org/en/latest/feature-store/training-serving.html) 

* [Feature Store End-to-End Demo](https://docs.mlrun.org/en/latest/feature-store/end-to-end-demo/index.html) 

