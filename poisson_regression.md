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
By modeling the *log* of the expected value, the actual prediction ($Y = e^{\text{linear equation}}$) is forced to be strictly positive.

This fundamentally changes how we interpret the coefficients:

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

* Two different chi-square tests
    * **Pearson statistic**: $$ X^2 = \sum_{i=1}^n \frac{(y_i - \hat{\mu}_i)^2}{\text{Var}(\hat{\mu}_i)} $$
        * *Notes*: Compares the observed counts ($y_i$) to the expected counts ($\hat{\mu}_i$) scaled by the variance. For a Poisson model, $\text{Var}(\hat{\mu}_i) = \hat{\mu}_i$. If the ratio of the Pearson statistic to the model's degrees of freedom ($X^2 / df$) is significantly greater than 1, it indicates overdispersion or a poor model fit.
    * **Deviance statistic**: $$ D = 2 \sum_{i=1}^n \left[ y_i \ln\left(\frac{y_i}{\hat{\mu}_i}\right) - (y_i - \hat{\mu}_i) \right] $$
        * *Notes*: Based on the likelihood ratio test, it compares the log-likelihood of your fitted model to a "saturated" model (a hypothetical model with a parameter for every observation that perfectly fits the data). As with the Pearson statistic, a residual deviance significantly larger than the residual degrees of freedom ($D / df > 1$) often indicates overdispersion or missing predictors.


Week 12: visualization and statistical testing
Week 13: finish up statistical testing and vis, start training
Week 14: finish presentation

