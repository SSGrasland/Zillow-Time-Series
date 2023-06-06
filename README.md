

# Using Zillow Housing Data to Forecast Home Values in Washington State

![Photo by Ashlynn Murphy](https://github.com/SSGrasland/Zillow-Time-Series/blob/main/Images/ashlynn-murphy-m8SJzZ8ID7Q-unsplash.jpg)

# Business Understanding
Our client Steady has had great success with their sustainable housing community in King County, WA. They are looking to build a new community elsewhere in the state of Washington. They want the community to be affordable to first time home owners so they are looking at the best zip codes to invest in with homes that are 500,000 USD or less in value. 

## The Problem 
Steady, a housing development agency, is unsure of where they want to build their next housing community. It has to be in the state of Washington and affordable to first time home owners, so home values need to be 500,000 USD or less. 

## The Solution 
Data from April 1996 to April 2018 from Zillow was used to build a time series model. The model was used to forecast the best zip codes to build in. 

### Metric: ROI
The metric used is return on investment. The top 5 zip codes with the best average 3 year ROI were isolated. Models were trained on the zipcode with the highest ROI and the model was used to forecast which top 3 zip codes would have the best ROI for the next year. 

# Data Understanding

#### Zillow Housing Data

This data represents median monthly housing sales prices for 265 zip codes over the period of April 1996 through April 2018 as reported by Zillow. The data was originally created by Zillow and obtained on Kaggle for the purpose of this project. 

Each row represents a unique home and its location and value. Each record contains location info, such as, city, zipcode, and metro and median housing sales prices for each month.

The dates will be used as the index value for doing time series analysis on monthly housing prices for the span April 1996 to April 2018. 

There are 14,723 rows and 272 variables:

- **RegionID**: Unique index, 58196 through 753844   
- **RegionName**: Unique Zip Code, 1001 through 99901   
- **City**: City in which the zip code is located    
- **State**: State in which the zip code is located     
- **Metro**: Metropolitan Area in which the zip code is located      
- **CountyName**: County in which the zip code is located       
- **SizeRank**: Numerical rank of size of zip code, ranked 1 through 14723       
- **1996-04 through 2018-04**: refers to the median housing sales values for April 1996 through April 2018, that is 265 data points of monthly data for each zip code  
- **Value**: refers to the median housing price

## Limiting the Dataset

For this project our client is specifically interested in Zip Codes in the state of Washington with a price ceiling of lower than 500,000USD

### Scrubbing Sale Values 
Sale values are missing from 2014-06 back. To deal with some missing values I will drop anything before 1998-04 so that the dataset spans exactly 20 years. Then any rows that are missing values will be dropped. I believe dropping the rows rather than filling in missing values with the mean is better in this instance because we want our data to remain accurate and it's less than 10% of the data. 

Later on in this notebook we will be limiting the data to span only the last five years. So the missing values before 2008-10 will not be as significant. 

## Exploring Return on Investments
Let's look at the 20 year, 10 year, 5 year, and 1 year ROI on each zip code in Washington. ROI is equal to the (current price - the original price/original)*100. 

Highest 20 year ROI: 230%  
Highest 10 year ROI: 66%   
Highest 5 year ROI: 115%   
Highest 1 year ROI: 35%    

It is expected that there will be a relationship between time and ROI. Typically real estate increases in value naturally with time, however, for some reason the return on investment for the 10 year ROI was lower than the 5 year ROI. This is most likely due to the 2008 housing crisis. If the housing crisis had not occurred we would expect the 10 year ROI to be around 180%. We will need to consider the impact of the 2008 housing crisis on our Time Series model. 

### Average one year ROI over the past 5 years 

The top 5 zip codes with the highest average one year ROI over the past 5 years are:
- 98168 = 27.78%
- 98146 = 25.96%
- 98043 = 25.77%
- 98294 = 25.35%
- 98251 = 25.20%

![Average_One_Year_ROI](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/avg_1_yr_ROI.png))

## Date Indexing 
When working with time series data in Python it is helpful to have the dates in the index. We will be modifying the dataset to have the dates indexed. 

#### Time-series Index Slicing for Data Selection

Given the 2008 Housing Bubble precedent, I have decided to slice the dates and start my analysis in April 2013. As we can see from the plot below around 2013 is when home prices start increasing again and displaying more normal behavior. This leaves us with a 10 year time window.

![Home Values](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/homevalue_2013.png))

# Data Analysis

### Top 5 Zipcode Analysis
The top 5 zip codes with the highest average one year ROI over the past 5 years are:
- 98168 = 27.78%
- 98146 = 25.96%
- 98043 = 25.77%
- 98294 = 25.35%
- 98251 = 25.20%

![Top 5 Zipcodes, Linegraph](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/top_5_zips_line.png))
![Top 5 Zipcodes, Box & Whisker](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/top_5_zips_bw.png))


Let's visualize each of our top zip codes and run a Dickey-Fuller test. This is a statistical test that tests for stationarity. Our null hypothesis in this case is that the time series is not stationary. So if the test statistic result is less than the critical value then the null hypothesis can be rejected and we can say the series is stationary. 

We can also see from our previous plot that all the zipcodes express an upward linear trend. We will need to attempt to detrend the series by differencing the data. Differencing can be used to remove a series dependence on time. It is performed by subtracting the previous observation from the current observation. 

![Top 5 Zipcodes, Differenced](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/zip_line_diff.png))

### Stationary Zip Code Analysis 

Now that we've differenced our top 5 zip codes and attempted to make them stationary, let's do some analysis. 

### ACF and PACF
ACF and PACF are tools we can use to help us find the order of AR, MA, and ARMA models. 

#### PACF
PACF, is a partial correlation, which can be explained by the amount of correlation between a variable and one lag of that variable which is not explained by lower order lags. 

The shaded area on the graph is our confidence interval. If the correlation drops into the confidence interval it means it is not statistically significant. 

#### ACF
ACF, is the correlation of the time series with a lagged version of itself. 

![ACF](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/ACF.png))
![PACF](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/PACF.png))

#### Results
- The ACF plot showed a strong correlation at 1 
- Geometric decay in ACF plot and PACF
- the PACF plot showed a strong correlation at 1 and 2 

This means that our time series could best be modeled using ARMA 
AR(p) = 2  
MA(q) = 1


## Modeling

### Train/Test Split

When doing a train/test split on a time series you cannot randomly split the data seeing as its time sensitive. Hence, in this case we will make our training data from our first 4 years of data and our testing data from our last year of data. 
![Train/Test Split](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/train-valid.png))

#### RMSE

Root mean square error (RMSE) is a commonly used measure for evaluating the quality of a model's prediction. It shows how far a prediction is from the measured true value by using Euclidean distance. 

The RMSE is computed for the naive model to compare against our later models. 

### Comparing Models
Let's find the best model by the RMSE of training and test data. The model with the best RMSE for training was the ARMA 21 model with a value of 665, the SARIMAX model had a value of 697.

The model with the best testing RMSE  was the SARIMAX model with a value of 2887. 

Since, the SARIMAX model performed better on our testing data this is the model we will use to forecast our time series. 

### Best Model: SARIMAX

Our SARIMAX model with an order of 0, 1, 3 and seasonal order of 0, 0, 0, 0 performed the best on our testing data. 

![SARIMAX](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/SARIMAX.png))

## Data Interpretation

### Forecasting Each Zip

The best model will be fit to each of our top 5 zip codes. Which will be used to forecast home prices for the next year. ROI will be calculated using the average forecast value versus the average house value for the last year of our data. 

### Predicted ROI of The Top Five Zip Codes   
Zipcode	Predicted ROI  
98146	18.630165   
98294	-0.095713    
98043	-22.763217     
98168	-29.933986     
98251	-35.449640    

![Predicted](https://github.com/SSGrasland/Phase4Project-/blob/main/Images/predicted.png))


## Recommendations and Conclusion

Steady, our client, wants to know which zipcode they should look at in Washington to develop their next housing community. 

The top three zip codes we recommend Steady to invest in are:

1. 98146, Seattle, WA which has a predicted 1 year ROI of 18.63%   
2. 98294, Sultan, WA which has a predicted 1 year ROI of -0.09%
3. 98043, Mountlake Terrace, WA which has a predicted 1 year ROI of -22.76%
 

### Future Work 
In the future it would be great to have more recent data to work with. This data is limited due to the housing market crash of 2008. It would be interesting to see what this data looks like once we have some distance from the crash. 

## Contact Me 
Thank you for looking at this project. If you have any further questions please contact me at sam.grasland@gmail.com
