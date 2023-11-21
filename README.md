# Mortality Prediction with Deep Learning
Priyonto Saha, Ente Kang, Gwen Eagle

## Introduction 
The ability to accurately predict mortality in the first 24 hours following admission to an Intensive Care Unit (ICU) is a tool that could provide significant benefit to individual patients and their families and the health care system overall. The intensive medical supervision in the ICU  provides a wealth of data that could be leveraged to inform the risk-benefit profile of medical interventions, evaluate the treatment options against palliative care and aid high-stakes decision making for clinicians and patient families. It would provide invaluable information to families as they emotionally and practically prepare for potential likely outcomes, and help them make time-sensitive decisions about end-of-life care. Moreover, mortality risk assessment can improve hospital-level planning and resource allocation, ensuring that hospitals are equipped in both personnel and medical equipment to meet the demands of a particular ICU patient load.

Thus, the objective of this paper is to understand the risk factors that predict in-hospital mortality and use these factors to build a neural network model that can accurately predict hospital death. The secondary objective is to determine if this neural network model has superior accuracy in predicting hospital death, compared to a logistic regression model based only on the APACHE IVa probabilistic prediction of in-ICU and in-hospital mortality for the patient, which utilizes the APACHE III score and other covariates, including diagnosis. 

## Data Engineering Process 
The dataset used includes records from 91713 unique patients from ICU units at hospitals around the world in Argentina, Australia, New Zealand, Sri Lanka, Brazil and over 200 hospitals in the United States, over an entire year, and consists of 186 variables, including demographic variables (i.e. age, ethnicity, BMI, gender), variables used in the calculation of the highest APACHE III score, comorbidities (i.e. AIDS, cirrhosis), lab gas values, vital signs, and lab values. The exploratory analysis began by classifying the data types of each column and viewing descriptive statistics by variable, followed by exploration of missing values. Boxplots were plotted for demographic variables versus the outcome to understand the demographic profile of patients at higher risk of mortality and to identify outliers. The exploratory analysis confirmed all observations were for a unique patient ID and thus the model did not have to account for correlation within individuals. Correlation matrices were constructed by class of variables  (demographic, APACHE III variables, lab gas values, vital signs and lab values) with the outcome variable (‘hospital death’) appended so as to assess multicollinearity within the same class and the relationship to the outcome. Pairwise association plots were constructed for hospital death to visualize relationship between features and the prediction target, hospital death. 

In preparation for the neural network and the logistic regression model, the data was processed by combining columns with the same keyword in their name and calculating the mean of those columns, while ignoring missing values, to account for multicollinearity, reduce variable missingness and improve model interpretation.  The manual feature selection process involved dropping several variables from the dataset, including identification numbers, gender, ethnicity, admission sources, stay type, ICU type readmission status, height, weight, APACHE APACHE III/II  admission diagnosis group and APACHE IVa probabilistic prediction of in-ICU and in-hospital mortality for the patient,  due to either irrelevance to the outcome,  circumventing  racial/gender bias, or multicollinearity. The variables included in the neural network model included age, BMI,  APACHE variables, comorbidities and the mean values of each lab measurement, including the APACHE lab values (i.e. mean potassium, mean glucose, mean temperature). Data missingness by each modified variable was calculated and variables with greater than 50% missing were dropped, including ‘lactate’, ‘fiO2’, ‘pH’, ‘pCO2’, ‘INR’, ‘bilirubin’, ‘APACHE urine output’ and ‘albumin’ . Correlation heat maps were reconstructed to confirm absences of multicollinearity and correlation for missingness between variables.  The data was split for training (80%) and testing (20%) with a random state equal to 42 to ensure reproducibility. The mean of ‘hospital death’ was checked in both datasets to ensure it was equal, and ‘ hospital death’ was identified as the outcome variable and converted into a numerical format. 

## Analysis 
A neural network model was selected to attempt to design a flexible function capable of modeling any level of complexity without predefined assumptions for the function form. Firstly, the parameters for the model were defined with epochs equal to 100, learning rate equal to 0.000001, batch size equal to 32. The neural network was defined with the following layers: a linear layer with 41 input features with 32 output, a leaky rectified linear unit (ReLU), a dropout layer with probability of dropout equal to 0.6 as a measure to prevent overfitting, a linear layer with 32 input and 16 output, another leaky rectified linear unit (ReLU) and another dropout, and one last linear layer from 16 input to 1 output with a final sigmoid activation function for a binary output. The Adam optimizer was employed to adjust the models parameters during training to minimize the loss and the loss function was defined with Binary Cross-Entropy Loss due to the binary nature of the outcome. The model was set to training mode and predictions and losses were computed for a forward pass. Subsequently, back propagation was performed to compute gradients and the weights and bias were updated based on the gradients computed. The model was then set to evaluation mode without the computation of gradients to evaluate accuracy on the test data.

Next, a logistic regression was used to determine the association between the APACHE IVa probabilistic prediction of in-hospital mortality and in-ICU mortality and hospital mortality. Logistic regression was used because the outcome is binary and provides the probability estimate of the outcome, mortality, given the predictor variables.  A confusion matrix, class report and AUC curve was computed to evaluate the model on test data. 5-fold cross-validation was used to split training data within both models to reduce overfitting and tune hyperparameters. Within the folds, iterative imputation was used for missing variables, followed by synthetic minority over-sampling technique due to the imbalanced nature of the outcome, and lastly feature scaling.

## Findings 
The data set contained 91733 unique observations with 7915 (0.0863%) patients experiencing an in-hospital death. The mean age of the data set was 62.3 years (SD: 16.8). Outliers were identified by the box plots and supported by the summary statistics for the variables. The variables with the highest degree of missingness were lab values (i.e. highest bilirubin during the first hour of unit stay was 92.2% missing) and a significant proportion of variables were greater than 50% missing.The pairwise plots revealed a linear association between weight and height and BMI, as expected. The correlation heatmaps for each variable class revealed correlations between readmission status and all demographic variables, hepatic failure and cirrhosis, and  the minimum and maximum variables for the same laboratory measurements, as expected. The results of the correlation heatmap were used to guide data pre-processing and variable selection, for a final correlation matrix that significantly reduced multicollinearity. After dropping variables that still had greater than 50%  missing, the final neural network model contained 41 variables. 

The neural network model validation yielded a stable accuracy score of  70% on the test set and 91.2-91.7% for each fold, indicating the appropriate selection of model parameters and features as the model is able to learn consistent patterns and generalizes well to unseen data. Prior to hyperparameter tuning, the model was predicting ‘no hospital death’ nearly 100% of the time, producing a high false positive rate and low model sensitivity, which was resolved by a lowered learning rate, using leaky ReLU rather than ReLU to avoid dying neurons, and another added layer. However, even with these adjustments the model overall had high specificity and low sensitivity. The logistic regression model using only the APACHE IVa probabilistic prediction of in-hospital mortality and in-ICU mortality determined an accuracy score of 0.86 and AUC of 0.64 on the test set.  The precision values and recall values for no death event and death event were 0.88 and  0.96, and 0.61, and 0.32,  respectively. The number of true positive, true negative, false positive and false negative samples, determined by the confusion matrix, were 1439, 22093, 3070 and 912, respectively. Thus, the logistic regression model also has high specificity but low sensitivity. 

## Conclusion 
Although the neural network model using more features yielded higher accuracy compared to the logistic regression model based only on APACHE prediction scores, both models had high specificity and severely low sensitivity. This could be partially attributable to the severe class imbalance in the data with only 0.0863% of patients experiencing death that, despite using techniques such as SMOTE and dropout layers to prevent overfitting, could not be entirely overcome. Future direction includes the use of more balanced data to ensure the models can learn to predict death, the application of regularization techniques to prevent overfitting, and more time dedicated to tuning hyperparameters and trying different activation functions and hidden layers.
