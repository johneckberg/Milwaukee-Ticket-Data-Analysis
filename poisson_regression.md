# Poisson Regression and Negative Binomial Regression

* Poisson regression is a type of Generalized Linear Model (GLM) used to model count data (non-negative integers) or rates. We can find the parameters via MLE
* Generalized linear models follow closely with standard linear regression, yet they allow for a non-linear link function to be applied to the linear combination of features and data. A commonly used example of a GLM is logistic regression.
* Unlike standard linear regression, glms often don't have a closed form and have to be optimized iteratively
* Poisson regression assumes the response variable Y has a Poisson distribution, and like other GLMS, assumes that the log of the expected value of this distribution can be modeled by a linear combination of unknown parameters
* [When to use negative binomial and Poisson regression](https://stats.stackexchange.com/questions/653727/when-to-use-negative-binomial-and-poisson-regression)
    * this recommends bootstrapping standard error
* Notes on [Poisson Regression](https://online.stat.psu.edu/stat462/node/209/)


## Why Not Standard Linear Regression (OLS)?

Since we are predicting a count (number of parking tickets), standard linear regression fails for a few reasons:

* **Negative Predictions**: A linear equation goes on forever. If a street has very few tickets, a linear model might predict a negative number (e.g., -2 tickets), which is physically impossible.
* **Non-Constant Variance (Heteroscedasticity)**: OLS assumes the variance of errors is constant across all predictions. With count data, the variance naturally scales with the mean (streets averaging 100 tickets per day will have much wider numerical swings than streets averaging 2 tickets).
* **Discrete Outcomes**: OLS predicts continuous numbers, but our actual data consists of discrete non-negative integers (0, 1, 2...).

## The Log Link and Interpreting Coefficients

To fix the negative predictions problem, Poisson and Negative Binomial regression use a log link function: 
$$ \ln(\text{Expected Counts}) = \beta_0 + \beta_1X_1 + \dots $$
By modeling the log of the expected value, the actual prediction ($Y = e^{\text{linear equation}}$) is forced to be strictly positive.

This changes how we interpret the coefficients:

* In standard OLS, a coefficient of $\beta = 2$ means "a 1-unit increase in X adds 2 to Y" (an additive effect).
* In a Poisson/NB model, the effect is multiplicative. If a coefficient $\beta = 0.2$, the effect on $Y$ is $e^{0.2} \approx 1.22$. A 1-unit increase in X increases the ticket count by roughly 22%.


## Assumptions

* It carries with it the standard Poisson assumption that events must be independent in the sense that the arrival of one event will not make another more or less likely
* Over and under dispersion
    * Overdispersion means that the actual covariance matrix for the observed data exceeds that for the specified model for $P(Y|x)$. For a Poisson distribution, the mean and the variance are equal. 
    * In practice, the data almost never reflects this fact and we have overdispersion in the Poisson regression model if (as is often the case) the variance is greater than the mean.
    * It would be good for me to go through and measure the variance of our data and see how well in general we can fit a poisson regression model to it in the first place
    * (notes on over dispersion)[https://biometry.github.io/APES//LectureNotes/2016-JAGS/Overdispersion/OverdispersionJAGS.html]
        * "Overdispersion also includes the case where none of your data points are actually $0$." This will be our data for sure 
        * overdispersion arises “naturally” if important predictors are missing or functionally misspecified (e.g. linear instead of non-linear)
    * how would we measure the variance here? over what data? do we assume tickets are evenly spaced out over the day? this doesnt make sense?

    
## Poisson vs. Negative Binomial

* I'm realizing we should switch from Poisson Regression to Negative Binomial Regression to handle the overdispersion and lack of constant average over the period. The Negative Binomial (NB) regression framework is specifically designed to relax the "constant average rate" (homogeneity) assumption that restricts the standard Poisson model. While we don't actually know the rate that tickets occur over the course of a day, we can make a safe assumption that it is not at all constant.
    * **Unobserved Heterogeneity (Varying Rates)**: NB assumes the rate is not constant. There is unobserved heterogeneity, meaning the rate varies from observation to observation even with identical predictors (models the background rate via Gamma distribution).
    * **Handling Clustering**: Because the rate fluctuates, NB naturally allows for "clustering" or "contagion" (e.g. if one ticket is given, the probability of a second nearby increases).
    * **Decoupling Mean and Variance**: In Poisson, $Var(Y) = \mu$. In Negative Binomial, $Var(Y) = \mu + \alpha\mu^2$, which nicely absorbs the extra variance in the data.

## Performance/Goodness of fit Measures

* Two different methods for Poisson, not sure if they generalize to Negative binomial?
    * **Pearson statistic**: $$ X^2 = \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{\text{Var}(\hat{\mu}_i)} $$
        * *Notes*: Compares the observed counts ($y_i$) to the expected counts ($\hat{\mu}_i$) scaled by the variance. For a Poisson model, $\text{Var}(\hat{\mu}_i) = \hat{\mu}_i$. If the ratio of the Pearson statistic to the model's degrees of freedom ($X^2 / df$) is significantly greater than 1, it indicates overdispersion or a poor model fit.
    * **Deviance statistic**: $$ D = 2 \sum_{i=1}^n \left[ y_i \ln\left(\frac{y_i}{\hat{\mu}_i}\right) - (y_i - \hat{\mu}_i) \right] $$
        * *Notes*: Based on the likelihood ratio test, it compares the log-likelihood of your fitted model to a "saturated" model (a hypothetical model with a parameter for every observation that perfectly fits the data). As with the Pearson statistic, a residual deviance significantly larger than the residual degrees of freedom ($D / df > 1$) often indicates overdispersion or missing predictors.

**Hard right tail means a large amount of overdispersion**

A Negative Binomial (NB) model may fail to capture overdispersion if the variance exceeds the quadratic structure
of the model

What category of days causes this extreme overdispersion?

## Measuring co-linearity in both our categorical and ratio features

* **Chi-Square Test (and its false alarms)**: Chi-square will capture any relationship between categorical features (both linear and non-linear). Since our GLMs mostly care about additive/linear overlap, Chi-square might trigger false alarms. Plus, with a large dataset, almost everything will flag as statistically significant anyway?
* **Cramer's V**: Cramer's V from my understanding is Pearson's R for categorical data. Because Chi-square values blow up as your sample size grows, they are hard to interpret as a metric of "how bad is the overlap." Cramer's V takes that Chi-square value and scales it between `0` (no association) and `1` (perfect association). If two categorical features have a super high Cramer's V, they are effectively feeding the model the exact same information, and we might want to drop one.
* **Pearson's R**: For our standard continuous/ratio features, just check the classic Pearson correlation matrix

## Testing

* we need to measure the significance of our predictors relationship with both ticket count and with eachother
* Types of visual analysis to build:
    * **Target Variable Distribution (Histogram)**: Plot a histogram of daily ticket counts (`total_tickets`). So we can see the right-skewed tail and visually confirm the overdispersion (i.e., checking if the variance looks huge compared to the mean).
    * **Categorical Features vs. Tickets (Box/Violin Plots)**: Plot `total_tickets` on the Y-axis and group by features like `WeekDay`, `Month`, or `is_holiday` on the X-axis.
    * **Continuous Features vs. Tickets (Scatter + KDE Smoothing)**: For weather data (`max_temp`, `precipitation`)
    * **Over-Time Trends (Line Chart with Rolling Averages)**: A 7-day or 30-day moving average of ticket counts plotted over time will smooth out the weekend dips and let us see the larger seasonality trends (e.g., summer peaks or pandemic-related drops?).

## models to train

* **Poisson Regression (Standard GLM)**
    * *Loss Function (Negative Log-Likelihood)*: $L(\mu, y) = \mu - y \ln(\mu)$ (ignoring the constant $\ln(y!)$ term, where $\mu$ is the expected count).
    * Baseline model, assuming mean equals variance.
* **Negative Binomial Regression (GLM)**
    * *Loss Function (Negative Log-Likelihood)*: $L(\mu, \alpha, y) = -\left[ y \ln\left(\frac{\alpha \mu}{1 + \alpha \mu}\right) - \frac{1}{\alpha} \ln(1 + \alpha \mu) + \ln \Gamma\left(y + \frac{1}{\alpha}\right) \right]$ (ignoring constant terms).
    * Relaxes the strict Poisson assumption by estimating a dispersion parameter ($\alpha$) to handle variance > mean. **As the dispersion parameter (number of successes) in a negative binomial approaches infinity, it converges to a Poisson distribution**
    * https://www.statsmodels.org/stable/generated/statsmodels.discrete.discrete_model.NegativeBinomial.html
* **Regularized GLMs (Ridge/L2 Penalty)**
    * *Loss Function*: Modifies the above objectives by adding an L2 penalty: $L_{\text{base}} + \lambda \sum \beta_j^2$.
    * Used if collinearity is severe (e.g., high Cramer's V or Pearson's R) to shrink coefficients and stabilize the model.
* **XGBoost (Poisson Objective)**
    * *Loss Function*: Uses the Poisson negative log-likelihood: $L(\hat{y}, y) = \hat{y} - y \ln(\hat{y})$ (where $\hat{y}$ is the predicted count). XGBoost calculates the gradient and hessian of this loss to build its trees.
    * its tree structure bypasses the more strict assumptions of GLMs. Maps complex non-linear patterns and feature interactions without manual feature engineering. by capturing these non-linearities, it eliminates issues with overdispersion and is more robust to collinearity than a GLM. Might be helpful depending on how the statistical testing on our features go?
    * Downsides: If our features are largely independent and additive, a simple GLM might actually be a much better representation of the true data-generating process. XGBoost loses pure model interpretability (no explicit $\beta$ coefficients to explain exactly how much one feature drives tickets), and can easily overfit if not tuned carefully. It might be nice to be able to explain in our presentation which of our features drive?

**The bi-modal nature of the distribution is almost fully explained by the Sunday variable**
If we were simply fitting the standard distribution via MLE, we would have an issue. However, since we are doing regression; we are really asking, "what is the formula that tells me where the mean should be for this specific data point?"

### Features:

- rolling average ticket count
- time lagged snow emergency (snow emergencies often last into the next day and are often declared the night before )
- day of week
- month of year
- binary holiday indicator
- is weekend also good (it might be better to use is_sunday?)

### Collinearity

For our continuous and ordinal features, we can use Pearson’s correlation (for linear relationships) and Spearman’s rank correlation (for monotonic but possibly non-linear relationships) to check for collinearity. 

Variance Inflation Factor (VIF) is another similar tool I just read about. It tells us how much the variance of a regression coefficient is inflated due to collinearity with other predictors. If we see a VIF much greater than 5 or 10, that’s a red flag for multicollinearity.

$VIF_j = \frac{1}{1 - R_j^2}$

For every single feature in the selected list, the process treats that feature as the target (y) and all the other features as the predictors (x)

However,  a lot of our best features are binary (like is_holiday, is_weekend) or cyclical (day of week, month). Standard collinearity tests wont play nice with these . For binary features, we could look at Cramér’s V. For cyclical features, it gets trickier.

#### Sine/Cosine Encoding for Cyclical Features

For things like day of week or month, we could potentially  use sine and cosine transforms to capture their cyclical nature (so Monday is next to Sunday, December is next to January, etc.). This encoding lets the model see the periodicity better than if we just did one hot encoding. we could still check Spearman’s correlation between these encoded features and the response variable, but the relationship might not be monotonic so correlation is just a rough guide.

https://skforecast.org/latest/faq/cyclical-features-time-series.html

#### Model Validation: Backtesting

For time series, classic cross-validation doesn’t work because it would leak future information into the past. Instead, we can use some form of backtesting (a.k.a. rolling or expanding window validation). This means training on a chunk of the past, testing on the next chunk, and repeating—so every test set is always after its training set. There’s a great [Kaggle notebook on backtesting](https://www.kaggle.com/code/cworsnup/backtesting-cross-validation-for-timeseries) for code examples.

we can see TimeSeriesSplit class from the scikit-learn library
Im reading that it makes more sense to use sliding window backtesting for relatively frequent data like our daily counts
https://www.uber.com/us/en/blog/omphalos/
Maybe we could do a sliding window over 1 year at a time (dropping the covid period?) 


#### MLFLow

If you want we can use mlflow to log our features and metrics. I feel like it makes repeated training runs easier to track. I'm happy to set training classes up for poisson and negative binomial regression

* https://mlflow.org/docs/latest/api_reference/python_api/mlflow.statsmodels.html
* https://mlflow.org/docs/latest/api_reference/python_api/mlflow.sklearn.html

I can set a wrapper around the training classes that let us pass in the dataframe and a list of features we want to use for that run and that way we can train in the background over the weekend?

Another question is how much data we want to use for this? How many years? do we want to drop the covid dip?

I can do outlier analysis after training 

Week 12: visualization and statistical testing
Week 13: finish up statistical testing and vis, start model selection/training
Week 14: finish presentation

I can do slides on the model and testing, you can do slides on the feature engineering/ selection? we could trade off on the viz slides?

