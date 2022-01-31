+++
title = "MLRun Feature Store" 
description = "Serverless real-time + offline feature store with scalable and high-performance data transformation layer"
weight = 5
+++

# Introducing the Iguazio feature store
Iguazio’s feature store helps to solve the hardest challenge of building a machine learning pipeline --- the feature engineering part.
The feature store provides:
- A robust transformation service
- A central hub for searching and managing your features
- Feature governance 

Core benefits of Iguazio’s feature store
- **Real time feature engineering** <br>
With Iguazio’s feature store, you can process real time events coming from streaming engines like Kafka, Kinesis, 
and others, and run transformations in real time. The real time transformation process is based on a scalable, fast, 
and serverless, function framework (Nuclio), along with an extremely fast key value database that is part of Iguazio’s 
platform. This is a must-have capability for building online ML use-cases such as fraud prediction, product recommendation, 
predictive maintenance, etc.
- **Automate the production deployment**<br>
Once you complete the feature exploration and have the transformation logic in place, deploying the transformation to 
production is as simple as running a one-line command. This command builds a real time serverless function with 
triggers to the streaming engine and a Rest endpoint, dramatically simplifying the production deployment.
- **Simple SDK for complex transformation** <br>
The feature store provides a simple python SDK that data scientists can easily use for creating all kinds of feature 
transformations. It empowers the data scientist to create a complex transformation without the help of data engineers. 
You can choose from various engines (spark, pandas, and storey – the real time engine). Once the transformation is set, 
MLRun stored the data both in parquet files in an object store for training, and in a key value database for online serving.
- **Seamless integration with model monitoring and serving**<br>
When using the Iguazio platform, you get an out-of-the-box model drift mechanism that is based on the actual features 
running in production, along with the statistics from the training dataset that are stored in the feature store. 
In addition, it stores the recent production data as a feature set, enabling you to run a training process on fresh 
data upon drift. The seamless integration between the feature store and the serving functions means that you can use 
the feature stats that are stored in the feature store for online imputing during the real time serving.
- **Features collaboration**<br>
The feature store comes with a friendly UI where you can easily search and reuse features. You can view the features 
meta data, statistics, transformation logic and more. You can also create and manage features and feature vectors from the UI.
- **Consistency between training and serving**<br>
The feature store provides a consistent view between the training and the serving data. The data ingestion job stores the data in both offline and online feature sets, ensuring that the same transformation logic runs for both training and serving. This eliminates the risk of a train/serve skew.

The Iguazio feature store fully supports the Kubeflow Pipeline/CI engine, including tracking the pipeline in Kubeflow.

<img src="/docs/external-add-ons/feature-store/mlrun/images/feature-store-kubeflow.png"
  alt="Training & Serving Solution"
  class="mt-3 mb-3 border border-info rounded">
