# Steps

## Josiah Mathews, John Eckberg

### Hypothesis

* What is the daily rate of parking tickets issued in Milwaukee, and which factors most strongly influence that rate (if any)? In particular, I propose to test whether day of week, month/seasonality, holidays, and precipitation are associated with higher or lower daily ticket counts, including by citation type.
    * Null Hypothesis: day of week, month/seasonality, holidays, and precipitation do not affect daily ticket counts.
    * Alternative Hypothesis: at least one temporal or environmental factor affects daily ticket counts.

## Dataset

* **Parking citations:** Milwaukee parking ticket records obtained through a Freedom of Information Act request (2014–2022), with fields including `ISSUENO`, `ISSUEDATE`, `LOCATIONDESC1`, and `VIODESCRIPTION`.
* **Holiday data:** U.S. holiday dates (2004–2021) pulled from Kaggle to identify atypical traffic/parking demand days.
* **Weather data:** Open-Meteo daily weather for Milwaukee, including maximum temperature, minimum temperature, precipitation, and snowfall.


### Data preprocessing

* Preprocessing steps:
    * Holiday Data
    * Weather Data
* Geotagging: 
    * Converting Street Addresses into Latitude and longitude points for visualization
    * Note: Exact address matching failed for many tickets for two main reasons: 1) manual data entry errors by parking enforcers (e.g., using approximate house numbers or "BLOCK"), and 2) the city's GIS Address Points dataset itself is incomplete, missing many valid, existing Milwaukee house addresses (verified via Google Maps). This justifies the use of a 100-block averaging fallback to map approximate ticket locations.



### Data analysis and visualization

* Heat Mapping (filter on features)
* Trend Analysis (Chi-square test across categories, Pearsons r for weather data)
* how does count change for different precipitation?
* how does count change for different days of the week?
* how does count change for different months?

### Features

* is holiday or not
* is snow emergency
* is weekend
* Heavy Rain or not
* precipitation amount
* Lagged ticket count from previous day
* Month/season

### Data modeling and prediction

* Poisson regression:
    * Poisson regression is a type of Generalized Linear Model (GLM) used to model count data (non-negative integers) or rates. We can find the parameters via MLE
    * Poisson regression assumes the response variable Y has a Poisson distribution, and like other GLMS, assumes that the log of the expected value of this distribution can be modeled by a linear combination of unknown parameters
    * It carries with it the standard Poisson assumption that events must be independent in the sense that the arrival of one call will not make another more or less likely
    * [When to use negative binomial and Poisson regression](https://stats.stackexchange.com/questions/653727/when-to-use-negative-binomial-and-poisson-regression)
        * this recommends bootstrapping standard error
    * Notes on [Poisson Regression](https://online.stat.psu.edu/stat462/node/209/)


### Results analysis

* Dealing with the assumptions of poisson regression
* Features we wanted but couldn't get
    * How city deficit and budget goals effects ticket count year over year
* make a note on the geocoding issues