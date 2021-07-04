# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, I built and optimized an Azure ML pipeline using the Python SDK and a custom Scikit-learn Logistic Regression model. I optimised the hyperparameters of this model using HyperDrive. Then, I used Azure AutoML to find an optimal model using the same dataset, so that I can compare the results of the two methods.

The following image illustrates the main steps I followed during the project.

![image](https://user-images.githubusercontent.com/45318647/124382752-abeda880-dce6-11eb-93e5-deb0815c23ac.png)

The whole exercise can be broken down into the following steps: 

Step 1: Set up the train script, create a Tabular Dataset from this set & evaluate it with the custom-code Scikit-learn logistic regression model.

Step 2: Creation of a Jupyter Notebook and use of HyperDrive to find the best hyperparameters for the logistic regression model.

Step 3: Next, load the same dataset in the Notebook with TabularDatasetFactory and use AutoML to find another optimized model.

Step 4: Finally, compare the results of the two methods and write a research report i.e. this Readme file.

## Summary

This dataset contains data about individuals applying for bank loans. The objective is to develop a model based on the information provided about each individual, to predict whether they will subscribe to a service or not.

The best performing model was found to be a voting ensemble with 91.5% accuracy.

## Scikit-learn Pipeline
**Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**
Parameter sampler

I specified the parameter sampler as mentioned below:
![image](https://user-images.githubusercontent.com/45318647/124383728-a8105500-dceb-11eb-9312-b60f341a8b65.png)

I chose discrete values with choice for both parameters, C and max_iter.
C is the Regularization while max_iter is the maximum number of iterations.

**What are the benefits of the parameter sampler you chose?**
RandomParameterSampling is one of the choices available for the sampler and I chose it because it is the faster and supports early termination of low-performance runs. If budget is not an issue, we could use GridParameterSampling to exhaustively search over the search space or BayesianParameterSampling to explore the hyperparameter space.

**What are the benefits of the early stopping policy you chose?**
![image](https://user-images.githubusercontent.com/45318647/124383918-98ddd700-dcec-11eb-89d5-24c9dd3a70d1.png)

evaluation_interval: This is optional and represents the frequency for applying the policy. Each time the training script logs the primary metric counts as one interval.
slack_factor: The amount of slack allowed with respect to the best performing training run. This factor specifies the slack as a ratio.

I selected the BanditPolicy stopping policy because it allows one to select a cut-off at which models reporting metrics worse than the current best model are terminated. This allows a relatively intuitive method to screen models, only retaining those with similar or better performance. This policy offers a little more flexibility than truncation and median stopping.

The best model parameters here were a C value of 0.01 and a max_iter value of 100. The model's accuracy was ~91.4%.

![image](https://user-images.githubusercontent.com/45318647/124385332-ac8c3c00-dcf2-11eb-8831-9c099d33abb1.png)


## AutoML
**Model and hyperparameters generated by AutoML.**

I defined the following configuration for the AutoML run:

![image](https://user-images.githubusercontent.com/45318647/124383956-db071880-dcec-11eb-938e-6278d8f68dbb.png)

- experiment_timeout_minutes=30 (This is an exit criterion and is used to define how long, in minutes, the experiment should continue to run. To help avoid experiment time out failures, I used the minimum of 30 minutes.)
- task='classification' (This defines the experiment type which in this case is classification.)
- primary_metric='accuracy' (I chose accuracy as the primary metric.)
- n_cross_validations=5 (This parameter sets how many cross validations to perform, based on the same number of folds (number of subsets). As one cross-validation could result in overfit, in my code I chose 5 folds for cross-validation; thus the metrics are calculated with the average of the 5 validation metrics.)

![image](https://user-images.githubusercontent.com/45318647/124385352-bdd54880-dcf2-11eb-9aa8-ee77b73c7f4a.png)


## Pipeline comparison
**Compare the two models and their performance. What are the differences in accuracy? In architecture? If there was a difference, why do you think there was one?**
The difference between the two models is trivial. The Hyperdrive model for the scikit learn pipeline (logistic regression) yields ana ccuracy of 91.4%, while the AutoML run (Voting Ensemble model) gives a slighlt better accuracy of 91.5%. In terms of the architecture, the two models are quite different. I feel that the AutoML model is actually better because of its AUC_weighted metric which equals to 94.6% and is more fit for the highly imbalanced data that we have here. If we were given more time to run the AutoML, the resulting model would certainly be much more better. And the best thing is that AutoML would make all the necessary calculations, trainings, validations, etc. without the need for us to do anything. Logistic regression (tuned with hyperdrive) effectively makes use of a fitted logistic function with a threshold to carry out binary classification. The voting ensemble classifier on the other hand (selected via autoML) makes use of a number of individual classifiers and, in this case, averages the class probabilities of each classifier to make a prediction.

## Future work
**What are some areas of improvement for future experiments? Why might these improvements help the model?**
Performing feature engineering on this data to construct more hypothesis based features would have helped in improving the predictive power of the model. Secondly, since the data is highly imbalanced, techniques for handling imbalanced data can be applied to get better predictions minimizing the false positives and false negatives. Accuracy is a bad metric to assess the classification model when built on top of imbalanced data. Metrics like weighted AUC, precision and recall give a much clearer picture of the model evaluation in such cases. Also increasing the cross validations from 5 to 10, accompanied by increase in experiment_timeout_minutes can also help in better generalisation.
