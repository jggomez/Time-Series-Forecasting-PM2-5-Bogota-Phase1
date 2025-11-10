# Time Series Forecasting: PM2.5 Levels in Bogotá

## **Authors:**
- Alejandra Valle Fernandez
- Juan Guillermo Gómez

## **1. Project Overview**

This project aims to implement a time series model to forecast daily PM2.5 levels in Bogotá for the next 7 days. PM2.5 was chosen due to its direct impact on public health and its strong correlation with the Air Quality Index (AQI). The analysis covers data selection, exploratory data analysis, stationarity transformations, autocorrelation analysis, and baseline model validation.

## **2. Dataset Selection and Characterization**

**Latin America Weather and Air Quality Data**

- **Source**: Kaggle (https://www.kaggle.com/datasets/anycaroliny/latin-america-weather-and-air-quality-data)
- **Content**: Two CSV files from the Open-Meteo API with weather and air quality data for various Latin American cities.
- **Frequency**: Daily data.
- **Parameters**: Country/city names, latitude/longitude, PM10, PM2.5, Carbon monoxide, Nitrogen dioxide, Sulphur dioxide, Ozone levels.

### **Key Characteristics (Bogotá PM2.5)**

| Characteristic | Description |
|---|---|
| Frequency | Daily |
| Main Variable | PM2.5 Concentration (µg/m³) |
| Observation Period | Continuous (August 2022 - April 2024), no missing data |
| Data Quality | No null values, few outliers after capping |

## **3. Phase 1 Objectives**

Upon completion of this phase, the student will demonstrate the ability to:

*   Conduct a comprehensive exploratory analysis of time series.
*   Identify fundamental patterns (trend, seasonality, cycles).
*   Apply transformations to achieve stationarity.
*   Implement and validate reference models.
*   Establish benchmark metrics for future phases.

## **4. Exploratory Data Analysis (EDA)**

### **Summary Statistics for PM2.5 (Bogotá)**

| Statistic | Value |
|---|---|
| Count | 626.00 |
| Mean | 21.55 µg/m³ |
| Std Dev | 9.06 µg/m³ |
| Min | 0.70 µg/m³ |
| 25% | 14.85 µg/m³ |
| 50% (Median)| 20.10 µg/m³ |
| 75% | 27.58 µg/m³ |
| Max (Capped)| 46.66 µg/m³ |
| Skewness | 0.61 (Right-skewed) |
| Kurtosis | 0.00 (Mesokurtic) |
| CV | 42.07% (Significant variability)|

**Conclusions:**

- The PM2.5 distribution is positively right-skewed, indicating occasional high pollution peaks. 
- The data shows significant variability relative to its average.
- Time series visualizations indicate irregular fluctuations with no clear seasonality but recurring increases in December, March, and April.

## **5. Seasonal Decomposition**

Decomposition into trend, seasonality, and residuals was performed using both additive and multiplicative models for periods of 7, 14, 21, and 28 days.

**Conclusions:**

- The **multiplicative model** consistently showed lower residual variance, indicating a better fit.
- A clear trend component was observed.
- **Seasonality was found to be weak** for both weekly (period=7) and monthly (period=30) cycles. The seasonal strength was very low (0.003 for period 7 and 0.007 for period 30).

## **6. Stationarity Analysis and Transformations**

Stationarity tests (Augmented Dickey-Fuller - ADF and Kwiatkowski-Phillips-Schmidt-Shin - KPSS) were applied to the original series and several transformations.

### **Original Series Stationarity**

- **ADF Test**: p-value = 0.2594 (Fail to reject H0 -> Not Stationary)
- **KPSS Test**: p-value = 0.0100 (Reject H0 -> Not Stationary)

**Conclusion**: The original PM2.5 series is **not stationary**, primarily due to a strong trend.

### **Transformations Applied:**

1.  **Simple Differencing (d=1)**
2.  **Seasonal Differencing (D=1, S=7)**
3.  **Logarithmic Transformation**
4.  **Logarithmic Transformation + Differencing (Log + Diff)**

### **Comparison of Stationarity Test Results**

| Transformation | ADF Statistic (lower = better) | KPSS Statistic (lower = better) |
| :--- | :--- | :--- |
| Original | -2.064 | 2.963 |
| Logarithmic | -2.534 | 2.960 |
| Differencing | -11.515 | 0.103 |
| Seasonal Differencing | -8.897 | 0.337 |
| **Log + Differencing** | **-11.879** | **0.083** |

**Conclusion**: **Logarithmic Transformation + Differencing** proved to be the most effective in achieving stationarity, as indicated by the most negative ADF statistic and the smallest KPSS statistic. This transformation addresses both trend and variance instability.

## **7. Autocorrelation Analysis**

ACF and PACF plots were generated for the **Log + Differenced** series to identify potential AR and MA components. The Ljung-Box test confirmed significant autocorrelation, indicating that the series is not white noise and contains patterns to be modeled.

### **Identified SARIMA Parameters:**

| Component | SARIMA Parameter | Identified Value | Identification Source | Reason/Justification |
| :--- | :--- | :--- | :--- | :--- |
| **Regular AR** | $p$ | $\mathbf{3}$ | PACF | Sharp cutoff after Lag 3. |
| **Regular MA** | $q$ | $\mathbf{2}$ | ACF | Sharp cutoff after Lag 2. |
| **Regular Differencing**| $d$ | $\mathbf{1}$ | Stationarity Tests | Differencing (Lag=1) removed trend. |
| **Seasonal Differencing**| $D$ | $\mathbf{0}$ | Seasonal Decomposition / ACF/PACF | No significant seasonality found at multiples of 7 after differencing. |
| **Seasonal AR** | $P$ | $\mathbf{0}$ | Seasonal PACF (Lags 7, 14, 21) | No significant spikes. |
| **Seasonal MA** | $Q$ | $\mathbf{0}$ | Seasonal ACF (Lags 7, 14, 21) | No significant spikes. |
| **Seasonal Period** | $S$ | $\mathbf{7}$ | Decomposition Analysis | Weekly cycle (although weak). |

## **8. Reference and Validation Models**

Baseline forecasting models (Naive, Seasonal Naive, Simple Moving Average - SMA, Drift) were evaluated across different transformations using a temporal train-test split (80/20) and time series cross-validation.

### **Baseline Model Performance (Test Set Evaluation)**

| Transformation | Model | MAE (µg/m³) | RMSE (µg/m³) | MAPE (%) |
| :--- | :--- | :--- | :--- | :--- |
| Original | Drift | 8.78 | 10.65 | 24.89 |
| Logarithmic Differencing | Naive | 9.78 | 11.78 | 27.44 |
| Logarithmic Differencing | Seasonal Naive | 11.34 | 13.58 | 32.62 |
| Logarithmic Differencing | SMA (k=7) | 10.64 | 12.61 | 30.04 |
| **Logarithmic Differencing** | **Drift** | **8.41** | **10.21** | **24.02** |

### **Cross-Validation Results (Mean RMSE and MAPE)**

| Model | Transformation | RMSE Mean | RMSE Std | MAPE Mean | MAPE Std |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Naive | Original | 8.87 | 2.06 | 28.93 | 7.91 |
| Drift | Original | 8.26 | 1.75 | 27.72 | 6.32 |
| Naive | Logarithmic | 8.87 | 2.06 | 28.93 | 7.91 |
| Drift | Logarithmic | 8.26 | 1.75 | 27.72 | 6.32 |
| Naive | Differencing | 8.87 | 2.06 | 28.93 | 7.91 |
| Drift | Differencing | 8.26 | 1.75 | 27.72 | 6.32 |
| Naive | Seasonal Differencing | 10.84 | 1.93 | 42.85 | 17.29 |
| Drift | Seasonal Differencing | 10.81 | 1.67 | 46.28 | 25.69 |
| Naive | Logarithmic Differencing | 8.87 | 2.06 | 28.93 | 7.91 |
| **Drift** | **Logarithmic Differencing** | **8.04** | **1.63** | **27.35** | **6.53** |

**Conclusion**: The **Drift model with Logarithmic Differencing** performed the best across both direct test set evaluation and cross-validation, achieving the lowest RMSE and competitive MAPE scores.

### **Residual Analysis of the Best Model (Drift with Logarithmic Differencing)**

- **Mean of residuals**: 7.16 (There's a positive bias, meaning the model consistently underestimates actual values by ~7.16 µg/m³).
- **Standard deviation**: 7.28
- **Skewness**: 0.075 (Close to ideal 0)
- **Kurtosis**: 0.554 (Close to ideal 0)
- **Normality (Shapiro-Wilk)**: p-value = 0.077 (Not strictly normal, but acceptable).
- **Autocorrelation (Ljung-Box)**: p-value = 0.0000 (Strong autocorrelation exists at lag 1).

**Conclusions:**

- The logarithmic transformation was effective in achieving good distribution properties (skewness, kurtosis).
- The main issues are the **positive bias** (underestimation) and **significant autocorrelation in residuals**, indicating that the simple Drift model, even with optimal transformation, is not fully capturing all patterns in the data. More sophisticated models will be needed to address these limitations.

## **9. Overall Conclusion**

This phase successfully characterized the PM2.5 time series for Bogotá, identified its non-stationary nature (strong trend, weak seasonality), and determined that Logarithmic Transformation + Differencing is the optimal pre-processing step. While baseline models provided a starting point, the residual analysis highlights the need for more advanced time series models (e.g., ARIMA, SARIMA) to accurately capture the remaining patterns and eliminate bias and autocorrelation in the forecasts.

## **10. Usage**

This project is presented as a Jupyter Notebook. You can execute the cells sequentially to replicate the analysis and visualizations. The notebook is designed to be self-contained, with data loaded directly from a URL.
