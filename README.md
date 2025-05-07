# covid_series_prediction: Respiratory disease analysis and predicting course of covid-19 infection for healthcare intervention

**Background** 
Respiratory disease are of various types and contagious. RSV, influenza, Covid infections require timely intervention and have similar symptoms. The datasets is published by Massachusettes government, department of public health and executive office of health and human services. In this study, a study of various types of respiratory infections was done to understand hospital visits, addmission rates, severity levels, mortality rates and overall healthcare burden in communities.

Sewer wastewater surveillance data samples were collected by CDC to detect covid infection in communities as early state indicator. The clinical data from hospitals comes with lag. The model was created to see if concentration levels of SARS-Cov2 in water could forecast disease spread in corelation with clinical data from hospitals. This was academic study to understand the possibilities. The data collection was at inception stage. Covid vaccination started in December 2020. The wastewater collection was a new initiative and testing facilities were setup. Samples were collected from sewers. Many communities have private sewers and entire population is not covered. While covid is not fatal after vaccine intervention, but SARS virus evolve over time due to mutation and need to be monitored for new strains.


**Goals**
Understand respiratory disease burden using clinical data and impact of covid and influenza vaccination. Understand if wasterwater surveillance data can help in predicting future infection rates using time series prediction model.


**Scope** 
Descriptive analysis: Respiratory disease burden quantification using patient ED visit and hospitalization data, severity levels, vaccination rates and wastewater covid concenrtation levels. Predictive analysis: Develop time series forecasting model to predict future disease trends to help policymakers and communities to prepare for potential surges.


**Prerequisites** 
Follow requirements.txt for all required packages. pmdarima requires older version of numpy==1.26.4 as the libaray has not been updated. It is used for finding best hyperparameters.


**Model selection**
Respiratory disease have yearly trend with peak season. Time series models require that series has stationarity.

Statsmodel SARIMAX library is used for modelling. SARIMAX library gives flexible choices to build ARIMA, SARIMA, SARIMAX models. 
It uses (pqd)(PQD)m values as parameters. Time series is decomposed to find seasonal, overall trend and residuals (errors). 
Augmented Dicky fuller ADF test is used to understand stationarity. The residuals are error components. They are assumed to be normally distributed, independent and unrelated to past data.

Autocorrelation function ACF detects relationship between current and past samples and number of lags that can be confidently forecasted. ACF coefficients give an inisght to choose the model based on coefficients.

Sinosoidal or exponetially decaying coefficients can be approximated with autoregressive models. Yt=C+θ Y(t-1)+ θ Y(t-2). Lags are indicated with p, θ are coefficients.

Others can be approxinated by moving averages. Yt=μ+Y(t-1)+ ϵ+Y(t-1)+ ϵ2. Lags are indicated by q, μ is mean,ϵ are error terms.

If samples are corelated with only 1 past sample, it cannot be approximated, its random walk.

Models with seasonality components have P,Q,D components with m samples in a season.

If models are Autoregressive, PACF partial coefficients are also observed as there can be collinearity across significant lags or past samples. 

Complex series have both p & q, AIC akaike information criterien is applied. It balances performance with complexity. Minimum lags with best performance are considered.

**Limitations**

The number of samples collected varied during the period. As the initiative ramped up and sewer collection sites added.

A live model would require steady feed of clean data with same number of samples, from same location and compared agaist the clinical data of that region to be precise. In this excercise, entire data of massachusettes was taken and comparison was made.

Most of the homes have private sewers. All the regions were not covered.

Multiple factors are involved: Population, exposure, proximity, isolation, age, immunity, vacination status, medical intervention, covid strain and associated severity. However, these factors were not considered in modelling.

Early medical intervention alter the outcomes and protect lives. The models can provide early indication to authorities and medical facilities to understand the preparedness.

**Data analysis**

Provenance: https://www.mass.gov/info-details/viral-respiratory-illness-reporting#wastewater-surveillance-reporting-

Dataset: Yearly season July 2023- July 2024 was taken as horizon for forecast.

ACF test indicated significant lag coefficient (q=5 significant coefficients : MA model), as well as sinosoidal lag ( p=3 significant coefficients: AR model), indicating a complex series. Hence PACF was observed.

The residuals were verified quantitatively and qualitatively.

The quantitative ljunbox null hypothesis assumes that residuals are normally distributed.

Qualitative analysis comprised Q-Q plot, KDE plot, correlogram and mean of residuals).

Hyperparameter search was performed using auto_arima library (pmdarima).

**Results**

**No seasonality model ARIMA**: p & q values for 1-52 week, d=0 was used by considering the series stationary and no seasonality. 

Weeks of forecasting window was considered in 45 weeks horizon based on analysis.The best AIC (1278.961579078176) was achieved with (2,0,1), (0,0,0,52). 

This meant it is an AR model, with 2 significant lag values. The 2 weeks is the forecasting window, in our horizon of 45 weeks.

**Seasonality model SARIMA**: Hyperparameters tested: p=1-52, q=1-52, d=0, P=0 as start, D=1 as mean/variance could varied across seasons. 

The best AIC value 974.944 was achieved with (3,0,0), (2,1,1,52). This model could forecast for 3 weeks.

SARIMAX model with wastewater data as exogenous data : The time series was taken from 2022-05 month. There were 107 samples. 46 were taken for horizon of prediction. 61 were used for training. No seasonality was considered, due to lack of number of seasons. Two exogenous features were considered. 

‘7 days average SARs-cov2 concentration’, ‘number of infected samples last 7 days’. Two models were created. One with ‘7 days average SARs-cov2 concentration’. Other by taking both.

Within this data following were ADF test outputs, indicating stationarity. ‘Covid numbers’: ADF Statistic: -2.282864431308097 p-value: 0.17755410281413658 '7day average of SARS-CoV-2 concentration: ADF Statistic: -0.8918661807719681 p-value: 0.7906391982055894

The parameter search was performed with p=0 to 30, q=0 to30, with no differencing as series were stationary. 

Seasonality was not considered. Both the models had similar best parameters (2,1,1), (0,0.0,0) was output by auto_arima. 

AIC was 271 and 273 respectively. Least of all the models. The diagnostic tests for residuals for were almost similar for both models. 

The residuals showed better Q-Q plots at 45 degrees, almost straight line. KDE plot was normally distributed. Ljunbox utility hypothesis was not rejected. Thus model is viable.

With data analysis, it was observed that covid infection cases with average SARS cov2 concentration levels in wastewater and count of infected samples can be used (as exogenous predictors) for creating predictive SARIMAX models.

**Model summary**

Models | Horizon |Window | MAPE | Exog. if considered | Seasonality

ARIMA | 46 weks |2 | 22.83 | None | None

SARIMA | 46 weeks |3 | 61.92 | None | Yes 52/season

SARIMAX1 | 46 Weeks |2 | 32.9 | SARS-cov2 conc.avg | None

SARIMAX2 | 46 Weeks |2 | 32.86 | SARS-cov2 conc.avg,infected |sample counts None

===============================================================================================
