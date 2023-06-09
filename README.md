<h2> 🌃 Analyzing Real Estate Data in St. Petersburg </h2>
<h3> 📙 Data source: </h3>

The [dataset](https://github.com/UtkovA/e2e_project/blob/main/spb.real.estate.archive.sample5000.tsv) is taken from [Yandex.Realty](https://realty.yandex.ru)

**Description of the dataset:**

Real estate listings for apartments in St. Petersburg and Leningrad Oblast from 2016 till the middle of August 2018.

Full EDA and visulization -> [EDA_and_visualization.ipynb](https://github.com/UtkovA/e2e_project/blob/main/EDA_and_visualization.ipynb)

Building a model -> [https://github.com/UtkovA/e2e_project/blob/main/building_model.ipynb](https://github.com/UtkovA/e2e_project/blob/main/EDA_and_visualization.ipynb)
 
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

Before cleaning:
![alt text](https://github.com/olgaselesnjova/E2E/blob/main/images/1.JPG)

After cleaning:
![alt text](https://github.com/olgaselesnjova/E2E/blob/main/images/1.JPG)

Basic graphs:
![alt text](https://github.com/olgaselesnjova/E2E/blob/main/images/1.JPG)

Correlation matrix:
![alt text](https://github.com/olgaselesnjova/E2E/blob/main/images/1.JPG)

3. Preprocessing
Data was preprocessed using mapper:
```
mapper = DataFrameMapper([([feature], SimpleImputer()) for feature in numeric_features] +\
                         [([feature], OneHotEncoder(handle_unknown='ignore')) for feature in nominal_features],
                             df_out=True)
```	

4. Scalers were used 
StandardScaler() is used to transform data to common format.
5. Building model and tuning hyperparameters
Different models were used and different parameters were tested to find the best bodel. The best result was shown by XGBregressor. 
Also, pipeline was used to combine all steps
```
pipeline = Pipeline(steps = [('preprocessing', mapper), 
                             ('scaler', StandardScaler()),
                             ('xgb', xgb.XGBRegresso (objective="reg:linear", random_state=42))])
```	
The result is following:
- Train
    - MAPE = 0.206
    - Accuracy = 0.709
- Test
    - MAPE = 0.231
    - Accuracy = 0.584

*For next steps RandomForestRegressor is used


<h3> 💻 How to install instructions and run the web-app with virtual environment </h3>
	
<h3> 📎 Information about Dockerfile </h3>

The **Dockerfile** starts with the Ubuntu 20.04 base image. The MAINTAINER command sets the author information for the image. 
- Then, the RUN command is used to update the package list on the Ubuntu image. 
- The COPY command is used to copy the content of the current directory to the /opt/gsom_predictor directory inside the Docker container. 
- The WORKDIR command sets the working directory inside the container as /opt/gsom_predictor. 
- The next RUN command installs the pip3 package manager for Python 3. 
- The final RUN command installs the dependencies listed in requirements.txt file using pip3. 
- The CMD command runs the app.py file using Python 3 inside the container.
	
<h3> ⚙️ How to open the port in a remote VM </h3>
	
We specify the port in a flask app in the script **app.py** by setting 
```
if __name__ == '__main__':
    app.run(debug = True, port = 5444, host = '0.0.0.0')
```
To open the remote VM port of our web application we need to use these lines:
```
sudo apt install ufw
sudo ufw allow 5444 
```
After that we can use applications such as **Postman** to check how our requests for API work.  

<h3> ⚓ How to run app using docker and which port it uses </h3>

Firstly we need to build containers and then run them: 
```
docker build -t <your login>/<directory name>:<version> .      # example: "docker build -t olgaselesnjova/e2e23:v.0.1 ."
docker run --network host -it <your login>/<directory name>:<version> /bin/bash
docker run --network host -d <your login>/<directory name>:<version>   
docker ps     # to show all running containers and info about them
docker stop <container name>    # from the list after docker ps
```
