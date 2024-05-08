# Anomaly Detection & Forecasting Using STL
Detecting fraudulent app usage data with STL and confidence intervals

## What is Seasonal Trend Decomposition using LOESS (STL)?
STL is a powerful technique used in time-series analysis to break down a given series to isolate components and understand underlying patterns. It breaks down the observed data into three fundamental components:

<p align="center">
    <img src="img\stl_example.png">
    <a href="https://www.statsmodels.org/dev/examples/notebooks/generated/stl_decomposition.html">https://www.statsmodels.org/dev/examples/notebooks/generated/stl_decomposition.html</a>
</p>

<b>Trend</b> - long-term movement in the data. It reflects whether the series is increasing, decreasing, or remaining relatively stable over time.<br>
<b>Seasonality</b> - repeating patterns that occur at fixed intervals (e.g., daily, monthly, or yearly) <br>
<b>Residuals (or Noise)</b> - random fluctuations or irregularities in the data that cannot be attributed to the other components. It includes measurement errors, outliers, and other unexpected variations.
In STL, LOESS stands for “Locally Estimated Scatterplot Smoothing,” which identifies local trends in the data by fitting regressions to small windows (neighborhoods) of data points. By focusing on local patterns rather than global trends, it is more robust against outliers and noisy data, helping in detecting true anomalies more accurately.

## How can STL be used to enhance forecasting?
Forecasting can be greatly affected by seasonality and noisiness of the data. Models like ARIMA tend not to perform well since the data can be fairly non-linear and irregular. By forecasting the trend, seasonality, and residuals separately and then adding the predictions together, you can create a forecast using ARIMA that better handles seasonality and noisiness since they were forecasted separately. The example will show this using pmdarima’s autoarima function, which will grid-search the best p, d, and q values to use for each component.

## Example: Detecting suspicious app activity using STL and forecasting with auto ARIMA
### Anomaly Detection
For this example, let’s say you have an app that you are tracking the daily active users for. You know that it has the highest number of users on Wednesday and the lowest number of users on Saturday and Sunday. You want to understand if certain spikes in daily active users are normal, statistically significant, or could even point to suspicious activity such as bots.

<p align="center">
    <img src="img\stl.png">
</p>

By breaking down the observed data into its three components using STL, we can see the patterns in the data more easily. Notably, there is a downtrend in the data towards the end of the graph, there is a clear seasonality to the data, and there are a few residuals that look pretty suspicious, especially the one towards the middle of the graph and the two towards the right. In order to be sure that we are detecting true anomalies, we will use confidence intervals. By fitting the trend line inside of the residuals (by subtracting the trendline by the overall mean of the trendline), we can create confidence intervals. By using confidence intervals at 1 standard deviation (90% confidence interval), 2 standard deviations (95% confidence interval), and 3 standard deviations (99.7% confidence intervals), we can classify the severity of the anomaly.

<p align="center">
    <img src="img\confidence_intervals.png">
</p>

Anomalies outside of the 90% confidence interval can be signals that daily active users is trending in an unusual way. Anomalies outside of the 95% confidence interval raise some stronger concerns around suspicious activity and irregular trends. Anomalies outside of the 99.7% confidence interval can be flagged as highly suspicious activity and could potentially point to bots if anomalies are above the confidence interval or disruption of service if anomalies are below the confidence interval.
When rerunning the STL with the anomaly detection algorithm, we can get a better idea of when the anomalies are happen with respect to the other components.

<p align="center">
    <img src="img\stl_anomaly_detected.png">
</p>

### Forecasting
ARIMA (Autoregressive Integrated Moving Average) is a model used for time series analysis, capturing temporal dependencies, trends, and seasonality. It assumes a linear relationship and may not capture complex nonlinear patterns. ARIMA is also very sensitive to noise, which can alter performance heavily. By running separate forecasts using ARIMA on each component from our decomposition, we can reduce the effect of residuals and get a better performing final model.

<p align="center">
    <img src="img\arima_forecast.png">
</p>

In this first example, the observed data was split into 80% train and 20% test. The top graph shows the model’s predictions of the last 20 days based on the test samples. The gray line is the observed data and the red dotted line is the model’s predictions.The bottom graph shows the forecast for the next 10 days. Based on how the model fit the test samples in the top graph, I was not confident that the model would predict the forecast accurately and wanted to use STL to get better performance.

<p align="center">
    <img src="img\stl_arima_forecast.png">
</p>

By creating predictions off of the trend, seasonality, and residual components individually and adding them up, the predictions came out far better. The predictions, depicted by the dotted red line, matched the test samples far more accurately, which gave me a lot more confidence in the predictions below. The confidence intervals were also adjusted based on the predictions from the individual components.

### Evaluation
The ARIMA-only model came out with a mean squared error (MSE) of 337.1796, whereas the model utilizing STL with ARIMA for forecasting had an MSE of 129.7460. Utilizing STL to enhance forecasting resulted in a significantly more performant model than what ARIMA could achieve alone.