# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
**In 1-2 sentences, explain the problem statement: e.g "This dataset contains data about... we seek to predict..."**
This dataset contains the data about different characteristics of customers of a bank (e.g. marital status, job specification, age, previous loan, defaulting status, education qualifications etc.) and we seek to predict wheather a customer applied for a fixed term deposit.

**In 1-2 sentences, explain the solution: e.g. "The best performing model was a ..."**
The best performing model was a Logistic Regression model with parameter value of C (Inverse of Regularization strength) and max_iter (Maximum number of iterations taken for the solvers to converge) as '0.02181554625496209' and '150' respectively. It performed with an accuracy of 0.9102446201949604.

## Scikit-learn Pipeline
**Explain the pipeline architecture, including data, hyperparameter tuning, and classification algorithm.**
The pipeline is created using HyperDriveConfig which requires an estimator, an early termination policy, a parameter sampler, a primary metric name, a primary metric goal and a value for maximum total runs. A parameter sampler is created using RandomParameterSampling which generates a set values (equal to value of maximum total runs) of C and max_iter to be used in child runs of experiment. For C, a continuous set of values ranging from 0.0005 to 1.0 is used and for max_iter, a discrete set of values is used which includes 50,100,150,200 and 250. BanditPolicy is used as early termination policy with evalution_interval as 5, slack_amount as 0.2 and delay_evalution as 5. This means if Run X is the currently best performing run with an accuracy of 0.9 after 5 intervals, then any run with an accuracy less than 0.7 (0.9 - 0.2) after 5 iterations will be terminated, and the delay_evaluation will delay the first termination policy evaluation for 5 sequences. An SKLearn estimator is used with train.py as training script for Scikit-Learn experiments which trains a Logistic Regression model on bank data (in which the characteristics are converted to numerical values with the help of clean data function defined in train.py which uses one hot encoding method) with varying sets of values of C and max_iter supplied by the parameter sampler (RandomParameterSampling in our case).

**What are the benefits of the parameter sampler you chose?**
The parameter sampler chosen is RandomParameterSampling which selects hyperparameter values randomly from the defined search space. RandomParameterSampling results in good results without consuming too much time.

**What are the benefits of the early stopping policy you chose?**
The early stopping policy chosen is BanditPolicy with evalution_interval as 5, slack_amount as 0.2 and delay_evalution as 5. This means if Run X is the currently best performing run with an accuracy of 0.9 after 5 intervals, then any run with an accuracy less than 0.7 (0.9 - 0.2) after 5 iterations will be terminated, and the delay_evaluation will delay the first termination policy evaluation for 5 sequences. This means I will not lose promising jobs and also the jobs with poor performance will be terminated early hence saving computation time and costs.

## AutoML
**In 1-2 sentences, describe the model and hyperparameters generated by AutoML.**
AutoML is configured through AutoMLConfig with experiment_timeout_minutes being 30 minutes ( i.e. the AutoML run will close after 30 minutes if not terminated beforehand ) , task being 'classification', primary_metric being 'accuracy', training_data being the whole dataset including 'features' and 'labels', label_column_name being the name of label column and n_cross_validations being 5 (i.e. the dataset will be split into 5 different dataset for cross validation purposes in order to make our model more confident). AutoML ran 53 iterations which resulted in VotingEnsemble pipeline as the best one with metric score ( i.e. accuracy) equals 0.9173. VotingEnsemble is an ensemble model which combines multiple models to improve machine learning results. It does so by predicting output on the weighted average of predicted class probabilities. So, the hyperparameters for VotingEnsemble pipeline are the ensemble_iterations and their respective weights. According to my code outputs, out of the 53 iterations ran by AutoML, iteration 46 ('XGBoostClassifier') , iteration 31 ('XGBoostClassifier'), iteration 0 ('LightGBM'), iteration 1 ('XGBoostClassifier'), iteration 51 ('XGBoostClassifier'), iteration 15 ('SGD') and iteration 22 ('LightGBM') were chosen to be the ensemble iterations and 0.14285714285714285, 0.14285714285714285, 0.14285714285714285, 0.14285714285714285, 0.14285714285714285, 0.14285714285714285 and 0.14285714285714285 were their respective weights.  

## Pipeline comparison
**Compare the two models and their performance. What are the differences in accuracy? In architecture? If there was a difference, why do you think there was one?**
The Scikit-learn pipeline ran with the help of HyperDriveConfig which used only a single model i.e. Logistic Regression model and gave an accuracy of 0.9102446201949604 where as AutoML generated VotingEnsemble as the best model which used multiple models which in turn resulted in improved predictive performance and gave an accuracy of 0.9173. So, there was a difference of approximately 0.007 in accuracy.
Logistic Regression models were fed with 2 arguments ( C (Inverse of Regularization strength) and max_iter (Maximum number of iterations taken for the solvers to converge) ) to vary the architecture and it had its best run with parameter value of C and max_iter as '0.02181554625496209' and '150' respectively.
The AutoML produced best results with VotingEnsemble model which chose 6 models ( out of 53 models ) with the help of Caruana ensemble selection algorithm with sorted ensemble initialization. The VotingEnsemble method chose the following:
(1) 'ensembled_iterations': '[46, 31, 0, 1, 51, 15, 22]'
(2) 'ensembled_algorithms': "['XGBoostClassifier', 'XGBoostClassifier', 'LightGBM', 'XGBoostClassifier', 'XGBoostClassifier', 'SGD', 'LightGBM']",
(3) 'ensemble_weights': '[0.14285714285714285, 0.14285714285714285, 0.14285714285714285, 0.14285714285714285, 0.14285714285714285, 0.14285714285714285, 0.14285714285714285]'
It is evident that difference in the accuracy is 0.007 and VotingEnsemble model given by AutoML performed better than the best LogisticRegression model given by HyperDriveConfig. The VotingEnsemble model implements the priciple of weighted average of predicted class probabilities ( soft voting ) and hence it results in giving more weight to the confident model. Also, AutoML alerted in the beginning that the data was imbalanced and had a bias towards one class. So possibly, VotingEnsemble covered for the weakness in individual models due to class imbalance and hence performed better than the best individual performing model in the ensemble ( i.e. iteration 46, 'XGBoostClassifier', accuracy : 0.9152959028831564). Whereas, Logistic Regression model being a single model might have had bad effect of class imbalance on its accuracy score.

## Future work
**What are some areas of improvement for future experiments? Why might these improvements help the model?**
Areas of improvement:
(1) Since their is class imbalance as pointed by AutoML, we can try weighted Logistic Regression by using class_weights hyperparameter.
(2) We can try to vary solver ( i.e. algorithm to use in the optimization problem) in Parameter Sampler. There is a possibility ( not certainly ) that 'sag' and 'saga' might give better results than default 'lbfgs' solver.
