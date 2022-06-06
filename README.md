                                                    Wafer Fault Detection Project

Project Link - https://waferfaultdetect.herokuapp.com/

Project Demo:-

![AnimationScreen](https://user-images.githubusercontent.com/61505882/129208003-54e4e06d-da1b-4325-9c5d-a28211601cee.gif)

Checking the prediction in Postman:-


![Postman](https://user-images.githubusercontent.com/61505882/129209909-00997a7a-9e38-4f1e-a4a8-8fc32fb27bdf.gif)


Problem Statement:-

The inputs of various sensors for different wafers have been provided. In electronics, a wafer (also called a slice or substrate) is a thin slice of semiconductor used for the fabrication of integrated circuits. The goal is to build a machine learning model which predicts whether a wafer needs to be replaced or not(i.e., whether it is working or not) based on the inputs from various sensors. There are two classes: +1 and -1. 

☑️	+1 means that the wafer is in a working condition and it doesn’t need to be replaced.

☑️	-1 means that the wafer is faulty and it needs to be replaced. 

Project Architecture:-

![image](https://user-images.githubusercontent.com/61505882/129191179-27c0d722-6ce5-496a-a64d-970473435262.png)


Data Description:-

The client will send data in multiple sets of files in batches at a given location. Data will contain Wafer names and 590 columns of different sensor values for each wafer. The last column will have the "Good/Bad" value for each wafer.
"Good/Bad" column will have two unique values +1 and -1.  

"+1" represents Bad wafer.
"-1" represents Good Wafer. 

Apart from training files, we also require a "schema" file from the client, which contains all the relevant information about the training files such as:
Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns, and their datatype.

![schematraining](https://user-images.githubusercontent.com/61505882/129371306-0a490dd9-d99b-48fe-87d9-5ad44844c8e2.gif)


 
Data Validation:-

In this step, we perform different sets of validation on the given set of training files.

1.	 Name Validation- We validate the name of the files based on the given name in the schema file. We have created a regex pattern as per the name given in the schema file to use for validation. After validating the pattern in the name, we check for the length of date in the file name as well as the length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder."

2.	 Number of Columns - We validate the number of columns present in the files, and if it doesn't match with the value given in the schema file, then the file is moved to "Bad_Data_Folder."

3.	 Name of Columns - The name of the columns is validated and should be the same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder".

4.	 The datatype of columns - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If the datatype is wrong, then the file is moved to "Bad_Data_Folder".

5.	Null values in columns - If any of the columns in a file have all the values as NULL or missing, we discard such a file and move it to "Bad_Data_Folder".

![RawFile](https://user-images.githubusercontent.com/61505882/129368623-05a0571e-5209-43f5-8068-34d851935c94.gif)


Data Insertion in Database:-
 
1) Database Creation and connection - Create a database with the given name passed. If the database is already created, open the connection to the database. 
  
2) Table creation in the database - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" based on given column names and datatype in the schema file. If the table is already present, then the new table is not created and new files are inserted in the already present table as we want training to be done on new as well as old training files.     

3) Insertion of files in the table - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".

![DBOperation](https://user-images.githubusercontent.com/61505882/129369676-1365b3c4-ffde-4b4e-82ba-c624abba7b23.gif)

 
Model Training:-

1) Data Export from Db - The data in a stored database is exported as a CSV file to be used for model training.

2) Data Preprocessing   
   a) Check for null values in the columns. If present, impute the null values using the KNN imputer.
   b) Check if any column has zero standard deviation, remove such columns as they don't give any information during model training.

3) Clustering - KMeans algorithm is used to create clusters in the preprocessed data. The optimum number of clusters is selected by plotting the elbow plot, and for the dynamic selection of the number of clusters, we are using "KneeLocator" function. The idea behind clustering is to implement different algorithms
   To train data in different clusters. The Kmeans model is trained over preprocessed data and the model is saved for further use in prediction.

4) Model Selection - After clusters are created, we find the best model for each cluster. We are using two algorithms, "Random Forest" and "XGBoost". For each cluster, both the algorithms are passed with the best parameters derived from GridSearch. We calculate the AUC scores for both models and select the model with the best score. Similarly, the model is selected for each cluster. All the models for every cluster are saved for use in prediction.

![ModelTraining](https://user-images.githubusercontent.com/61505882/129370319-15c83f34-942c-43b0-aef7-2b8c6933a3a4.gif)



![TrainingSucessful](https://user-images.githubusercontent.com/61505882/129366047-f616fa52-6df4-4067-8398-0cba7816bf3f.JPG)

 
 
 
Prediction Data Description:-
 
Client will send the data in multiple set of files in batches at a given location. Data will contain Wafer names and 590 columns of different sensor values for each wafer. 
Apart from prediction files, we also require a "schema" file from client which contains all the relevant information about the training files such as:
Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns and their datatype.

![schemaprediction_1](https://user-images.githubusercontent.com/61505882/129373317-91191dc4-c285-46a2-8664-4a2dffd6f690.gif)

 
Data Validation:-

In this step, we perform different sets of validation on the given set of training files.

1) Name Validation- We validate the name of the files on the basis of given Name in the schema file. We have created a regex pattern as per the name given in schema file, to use for validation. After validating the pattern in the name, we check for length of date in the file name as well as length of time in the file name. If all the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder". 

2) Number of Columns - We validate the number of columns present in the files, if it doesn't match with the value given in the schema file then the file is moved to "Bad_Data_Folder". 

3) Name of Columns - The name of the columns is validated and should be same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder". 

4) Datatype of columns - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If dataype is wrong then the file is moved to "Bad_Data_Folder". 

5) Null values in columns - If any of the columns in a file has all the values as NULL or missing, we discard such file and move it to "Bad_Data_Folder".  

![schemarawvalidation_1](https://user-images.githubusercontent.com/61505882/129377100-406338f7-18b9-432b-90b7-9b2686190501.gif)


Data Insertion in Database:-

1) Database Creation and connection - Create database with the given name passed. If the database is already created, open the connection to the database. 

2) Table creation in the database - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" on the basis of given column names and datatype in the schema file. If table is already present then new table is not created, and new files are inserted the already present table as we want training to be done on new as well old training files.     

3) Insertion of files in the table - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns, the file is not loaded in the table and is moved to "Bad_Data_Folder".


![dboperationfile](https://user-images.githubusercontent.com/61505882/129378620-368588ca-cbfb-40f8-90ca-8e1b81784147.gif)



Prediction:-
 
1) Data Export from Db - The data in the stored database is exported as a CSV file to be used for prediction.

2) Data Preprocessing    
   a) Check for null values in the columns. If present, impute the null values using the KNN imputer.
   b) Check if any column has zero standard deviation, remove such columns as we did in training.

3) Clustering - KMeans model created during training is loaded, and clusters for the preprocessed prediction data is predicted.

4) Prediction - Based on the cluster number, the respective model is loaded and is used to predict the data for that cluster.

5) Once the prediction is made for all the clusters, the predictions along with the Wafer names are saved in a CSV file at a given location and the location is returned to the client.

![predictioncode](https://user-images.githubusercontent.com/61505882/129380668-4bad7440-f45d-45a5-9c01-f530f49bed3e.gif)


![PredictionSucess](https://user-images.githubusercontent.com/61505882/129366204-79bee388-c0c7-489a-9525-96a0e187b65b.JPG)

 
Deployment

We will be deploying the model to the Heroku Cloud Platform
                                                                     

Now let’s see the Wafer fault Detection project folder structure.

The file that is required to deploy the application in Heroku:-

1)requirements.txt

2)Procfile

3)runtime.txt

![image](https://user-images.githubusercontent.com/61505882/129191726-10e266d2-b0aa-4d7b-92f3-852c8ef75d9c.png)

requirements.txt file consists of all the packages that you need to deploy the app in the cloud.

![image](https://user-images.githubusercontent.com/61505882/129192502-86181081-addf-4d08-80a7-d5478ea11784.png)

app.py is the entry point of our application, where the flask server starts and then we will be making predictions.

![image](https://user-images.githubusercontent.com/61505882/129192931-0bc9e1a7-1819-4f70-b75b-2f053248e645.png)

This is the PredictfromModel.py file where the predictions and  we are giving input to the model.

![image](https://user-images.githubusercontent.com/61505882/129201700-35872883-b3a4-48d3-aa1c-4d9c3861fec3.png)


![procfile](https://user-images.githubusercontent.com/61505882/129202225-391adc4c-8d5d-4ed7-9849-f26b8e86a134.jpg)

Procfile :- It contains the entry point of the app.


![image](https://user-images.githubusercontent.com/61505882/129202472-9d5399b5-960e-48cb-907a-a57138945702.png)

runtime.txt:- It contains the Python version number.





