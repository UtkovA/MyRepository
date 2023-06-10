<h2> :house: Analyzing Real Estate Data in St. Petersburg </h2>
<h3> :mortar_board: Data source: </h3>

The [dataset](https://github.com/UtkovA/e2e_project/blob/main/spb.real.estate.archive.sample5000.tsv) is taken from [Yandex.Realty](https://realty.yandex.ru)

:notebook_with_decorative_cover: **Description of the dataset:**

Real estate listings for apartments in St. Petersburg and Leningrad Oblast from 2016 till the middle of August 2018.

:pencil: Full EDA and visulization -> [EDA_and_visualization.ipynb](https://github.com/UtkovA/e2e_project/blob/main/EDA_and_visualization.ipynb)

:pencil: Building a model -> [Building_model.ipynb](https://github.com/UtkovA/e2e_project/blob/main/EDA_and_visualization.ipynb)
 
1. The dataset was analyzed with basic statistical tools and visualization
2. Cleaning
Dataset conations a lot of errors and outliers which were removed using several conditions:
```
sell_df_cleaned = sell_df_spb[~((sell_df_spb.price_per_sq_m/sell_df_spb.house_price_sqm_median) > 5)]
sell_df_cleaned = sell_df_cleaned[(sell_df_cleaned['last_price'] < 100000000) & (sell_df_cleaned['last_price'] > 2000000)]
sell_df_cleaned = sell_df_cleaned[~((sell_df_cleaned.price_per_sq_m > 500000) 
                                     & ((sell_df_cleaned.house_price_sqm_median < 200000) 
                                        | (sell_df_cleaned.house_price_sqm_median == sell_df_cleaned.price_per_sq_m)))]
sell_df_cleaned = sell_df_cleaned[~((sell_df_cleaned.price_per_sq_m < 38000) 
                               & (sell_df_cleaned.house_price_sqm_median/sell_df_cleaned.price_per_sq_m >= 2))]
sell_df_cleaned = sell_df_cleaned[~((sell_df_cleaned.price_per_sq_m < 30000) 
                                          & (sell_df_cleaned.price_per_sq_m == sell_df_cleaned.house_price_sqm_median))]

#Based on quartiles
sell_df_cleaned = sell_df_cleaned.fillna(sell_df_cleaned.median())
sell_df_cleaned = sell_df_cleaned[(sell_df_cleaned['floor'] <= 26)]
sell_df_cleaned = sell_df_cleaned[(sell_df_cleaned['living_area'] < 212)]
sell_df_cleaned = sell_df_cleaned[(sell_df_cleaned['kitchen_area'] < 67)]
sell_df_cleaned = sell_df_cleaned[(sell_df_cleaned['area'] < 342)]
```

Before cleaning:\
![alt text](https://github.com/UtkovA/e2e_project/blob/main/images/e2e_2.png) { width: 200px; }

After cleaning:\
![alt text](https://github.com/UtkovA/e2e_project/blob/main/images/e2e_3.png) { width: 400px; }

Basic graphs:
![alt text](https://github.com/UtkovA/e2e_project/blob/main/images/e2e_1.png)

Correlation:\
![alt text](https://github.com/UtkovA/e2e_project/blob/main/images/e2e_4.png)

3. Preprocessing

Data was preprocessed using mapper:
```
mapper = DataFrameMapper([([feature], SimpleImputer()) for feature in numeric_features] +\
                         [([feature], OneHotEncoder(handle_unknown='ignore')) for feature in nominal_features],
                             df_out=True)
```	

4. Scalers were used\
StandardScaler() is used to transform data to common format.
5. Building model and tuning hyperparameters\
Different models were used and different parameters were tested to find the best bodel. The best result was shown by XGBregressor.
```
grid_search.best_params_
{'xgb__learning_rate': 0.3, 'xgb__max_depth': 7, 'xgb__n_estimators': 450}
```

Also, pipeline was used to combine all steps
```
pipeline = Pipeline(steps = [('preprocessing', mapper), 
                             ('scaler', StandardScaler()),
                             ('xgb', xgb.XGBRegressor (objective="reg:linear", random_state=42))])
```	
The result is following:
- Train
    - MAPE = 0.206
    - Accuracy = 0.709
- Test
    - MAPE = 0.216
    - Accuracy = 0.647

<h3> :computer: Virtual environment, Docker and Postman </h3>

The next steps was about virtual connection and setting up prediction model not on local server:
6. To create "virtual machine" using Yandex.Cloud
7. To create Flask project and pickle files with the model and scalers
8. To connect Flask project with github repository
9. To run code on remote machine\
To do it we specified a port in a script <app.py> which is running on our virtual machine and open the port:
```
if __name__ == '__main__':
    app.run(debug = True, port = 5444, host = '0.0.0.0')

sudo apt install ufw
sudo ufw allow 5444 
```	
10. To check connection using postman (Show predicted price based on parameters)
11. To install docker on "Virtual machine" and then to set up it
12. To set up virtual environment on the "virtual machine"
```
sudo apt install python3.8-venv
python3 -m venv env
```
13. To install libraries on our virtual environment
14. To set up requrements files\
This file helps to download all necessary libraries
15. To create Dockerfile
```
FROM ubuntu:20.04
MAINTAINER Alexander Utkov
RUN apt-get update -y
COPY . /opt/gsom_predictor
WORKDIR /opt/gsom_predictor
RUN apt install -y python3-pip
RUN pip3 install -r requirements.txt
CMD python3 app.py
```
16. To create Docker image
```
docker build -t utkova/e2e_test:v.0.1 . #Names and versions are different for each user
docker images  # Show all Docker images
```
17. To run our model using Docker
```
docker run --network host -d utkova/e2e_test:v.0.1 #Names and versions are different for each user
```
18. To check the connetction using Postman
19.To push Docker image to DockerHub
```
docker push utkova/e2e_test:v.0.1 #Names and versions are different for each user
```

<h3> :rocket: Conclusion </h3>
THe whole process helps to understand how to run your model or your code using virtual environment and machines and show the basis steps which is used by majority of companies aroud the world. 

Made by:\
**Utkov Alexander**
