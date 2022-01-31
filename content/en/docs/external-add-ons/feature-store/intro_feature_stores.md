+++
title = "Introduction to Feature Stores" 
description = "An overview of feature stores in ML"
weight = 5
+++


## An overview of feature stores in ML

A feature store functions as a single pane of glass for generating, sharing, and analyzing all available features across the organization. It is a repository where anyone on an ML team can easily find previously-created features. But a feature store is much more than just a repository. It is also a data transformation service where users can manipulate raw data and store the data as features, ready to be used immediately by any machine learning model. This service significantly reduces duplicate work, which accelerates time to production and reduces complexity. 

## Seamless feature engineering


Feature engineering is the process of generating new features (for example, comparing a person’s current spending to the average amount spent over the last 3 months). These features must be consistent for model training with historical data as well as model prediction with real-time data. 

Feature consistency requires significant engineering effort and leads to model inaccuracy when not met. Additionally, monitoring solutions must be built to track features and results and send alerts of data or model drift.

Consider the spending example mentioned above. Creating such a feature is easy if you have the full dataset in training. However, in serving, one needs to calculate this feature in an online/real-time manner. 

The “brute-force” method is to have an ML engineer create an online pipeline that re-implements all the feature calculations done in the offline process. This method is time-consuming, error-prone, and very difficult to maintain over time, leading to lengthy deployment time. 

As you scale your MLOps, the “brute-force” method requires you to increase the number of data engineers and data scientists to create, implement, and monitor the necessary features. Large corporations that deal extensively with AI as part of their core value have built their own feature stores. Leaner teams need to find their own solution, and luckily the market for feature store solutions is starting to mature.

A feature store creates an easy way to create in-training features that are deployable to serving without defining all the “glue” code. 
Feature stores are comprised of the following:

* __Features__: An individual measurable property or characteristic of a phenomenon being observed. This may be raw data (e.g., transaction amount, image pixel, etc.) or a derivation from one or more features (e.g., deviation from average, pattern on image, etc.).

* [__Feature set__](https://docs.mlrun.org/en/latest/feature-store/feature-sets.html): A grouping of features. The grouping is done by setting the entity key or set-of keys (e.g., a transaction may be grouped by userID performing the transfer or by the deviceID used to perform the transaction). You can ingest data to a feature set.

* __Execution graph__: A set of operations performed on the data during ingestion. The graph contains steps which represent data sources, targets, data transformations ,and data enrichment. 

* [__Feature vector__](https://docs.mlrun.org/en/latest/feature-store/feature-vectors.html): A set of features, taken from one or more feature sets, defined prior to model training. Feature vectors serve as the input to the model training process. During model serving, the feature values in the vector are obtained from an online service. 

