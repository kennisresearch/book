## Bayesian Information Criterion (BIC)

In inferential statistics, we compare model selections using $p$-values or adjusted $R^2$. Here we will take the Bayesian propectives. We are going to discuss the Bayesian model selections using the Bayesian information criterion, or BIC. BIC is one of the Bayesian criteria used for Bayesian model selection, and tends to be one of the most popular criteria. 


### BIC

The Bayesian information criterion, BIC, is defined to be
$$ \text{BIC} = -2\ln(\text{likelihood}) + (p+1)\ln(n), $$
where $n$ is the number of observations in the model, and $p$ is the number of predictors. That is, $p+1$ is the number of total parameters (also the total number of coefficients, including the intercept) in the model.

Similar to AIC, the Akaike information criterion, the model with the smallest BIC score is preferrable. The above formula can be re-expressed using the model $R^2$ that we are familiar with:
$$ \text{BIC} = n\ln(1-R^2)+(p+1)\ln(n), $$
with the same $n$ and $p$. From this expression, we see that adding more predictors, that is, increasing $p$, will result in larger $R^2$, which leads to a smaller $\ln(1-R^2)$ in the first term of the BIC expression. While larger $R^2$ means better goodness of fit of the model, too many predictors may result in overfitting the data. Therefore, the second term $(p+1)\ln(n)$ is added in the BIC expression to penalize models with too many parameters. When $p$ increases, the second term increases as well. This provides a trade-off between the goodness of fit given by the first term and the model complexity represented by the second term.

### Backward Elimination with BIC

We will use the kids' cognitive score data set `cognitive` as an example. We first read in the data set from Gelman's website and transform the data types of the two variables `mom_work` and `mom_hs`, as what we did in the previous sections.


```r
# Load the library in order to read in data from website
library(foreign)    

# Read in cognitive score data set and process data tranformations
cognitive = read.dta("http://www.stat.columbia.edu/~gelman/arm/examples/child.iq/kidiq.dta")

cognitive$mom_work = as.numeric(cognitive$mom_work > 1)
cognitive$mom_hs =  as.numeric(cognitive$mom_hs > 0)
colnames(cognitive) = c("kid_score", "hs","IQ", "work", "age")
```

We will start with the full model, with all possible predictors: `hs`, `iq`, `work`, and `age`. We will drop one variable at a time and record all BIC scores. Then we will choose the model with the smallest BIC score. We will repeat this process until none of the models yield a decrease in BIC. We use the `step` function in R to perform the BIC model selection. Notice the default `k` argument in the `step` function is `k=2`, which is for the AIC score. For BIC, `k` should be `log(n)` correspondingly.


```r
# Compute the total number of observations
n = nrow(cognitive)

# Full model using all predictors
cog.lm = lm(kid_score ~ ., data=cognitive)

# Perform BIC elimination from full model
cog.step = step(cog.lm, k=log(n))   # penalty for BIC rather than AIC
```

```
## Start:  AIC=2541.07
## kid_score ~ hs + IQ + work + age
## 
##        Df Sum of Sq    RSS    AIC
## - age   1     143.0 141365 2535.4
## - work  1     383.5 141605 2536.2
## - hs    1    1595.1 142817 2539.9
## <none>              141222 2541.1
## - IQ    1   28219.9 169441 2614.1
## 
## Step:  AIC=2535.44
## kid_score ~ hs + IQ + work
## 
##        Df Sum of Sq    RSS    AIC
## - work  1     392.5 141757 2530.6
## - hs    1    1845.7 143210 2535.0
## <none>              141365 2535.4
## - IQ    1   28381.9 169747 2608.8
## 
## Step:  AIC=2530.57
## kid_score ~ hs + IQ
## 
##        Df Sum of Sq    RSS    AIC
## <none>              141757 2530.6
## - hs    1    2380.2 144137 2531.7
## - IQ    1   28504.1 170261 2604.0
```

In the summary chart, the `AIC` should be interpreted as BIC, since we have chosen to use the BIC expression where $k=\ln(n)$.

From the full model, we predict the kid's cognitive score from mom's high school status, mom's IQ score, mom's work status and mom's age. The BIC for the full model is 2541.1 <!--(from the row starting with `<none>`).-->

At the first step, we try to remove each variable from the full model. From the summary statistics, we can see that removing variable `age` results in the smallest BIC. But if we try to drop the `IQ` variable, this will increase the BIC, which implies that `IQ` would be a really important predictor of `kid_score`. Comparing all the results, we drop the `age` variable at the first step. After dropping `age`, the new BIC is now 2535.4.

At the next step, we see that dropping `work` variable will result in the lowest BIC, which is 2530.6. Now the model has become
$$ \text{score} \sim \text{hs} + \text{IQ} $$

Finally, when we try dropping either `hs` or `IQ`, it will result in higher BIC than 2530.6. This suggests that we have reached to our best model, which predicts kid's cognitive score using mom's high school status and mom's IQ score. 

However, using the adjusted $R^2$, the best model would be the one including not only `hs` and `IQ` variables, but also mom's work status, `work`. In general, using BIC leads to fewer variables for the best model compared to using adjusted $R^2$ or AIC.

We can also use the `BAS` package to find the best BIC model without taking the stepwise backward process.

Here we set the `modelprior` argument as `uniform()` to assign equal prior probability for each possible model.

The `logmarg` information inside the `cog.bic` summary list records the log-marginal likelihood of each model, which is proportional to negative BIC. We can use this information to retreat the model with the largest log-marginal likelihood, which corresponds to the model with the smallest BIC. 


```r
# Find the index of the model with the largest logmarg
best = which.max(cog.bic$logmarg)

# Retreat the variables that are in the best model, with 0 as the index of the intercept
bestmodel = cog.bic$which[[best]]
bestmodel
```

```
## [1] 0 1 2
```

```r
# Create a indicator vector indicating which variables are used in the best model
bestgamma = rep(0, cog.bic$n.vars) # Create a 0 vector with the same dimension of the number of variables in the full model
bestgamma[bestmodel + 1] = 1  # Change the indicator to 1 where variables are used
bestgamma
```

```
## [1] 1 1 1 0 0
```

From the indicator vector `bestgamma` we see that only the intercept (indexed as 0), mom's high school status variable `hs` (indexed as 1), and mom's IQ score `IQ` (indexed as 2) are used in the best model, with 1's in these indexes.


### Coefficient Estimates Under Reference Prior for Best BIC Model

The best BIC model can be set up as follows:
$$ \text{score}_i = \beta_0 + \beta_1(\text{hs}_i - \bar{\text{hs}})+\beta_2(\text{IQ}_i-\bar{\text{IQ}})+\epsilon_i, $$
where $\bar{\text{hs}}$ and $\bar{\text{IQ}}$ are the centers of the `hs` and `IQ` variables respectively. It is common to apply the reference prior again, and to compare with the results that we have obtained previously for the full model. In this case, the mean of $\beta_0$ we obtain later will be the mean of the kid's cognitive scores, or $\bar{\text{score}}$. Here we assume the joint prior of $\beta_0,\ \beta_1,\ \beta_2$ is
$$ \pi(\beta_0,\beta_1,\beta_2~|~\sigma^2) \propto 1, $$
and using the hierachical model, the prior of $\sigma^2$ is set to be
$$ \pi(\sigma^2)\propto \frac{1}{\sigma^2}. $$

Under this reference prior, BIC is proportional to the log-marginal likelihood, and therefore, we can use argument `prior = BIC` in the `bas.lm` function to obtain the statistics of these coefficients, when we impose the model in the `bas.lm` function to use only the variables shown in the best model.

```r
# Fit the best BIC model by imposing which variables to be used using the indicators
cog.bestbic = bas.lm(kid_score ~ ., data = cognitive,
                     prior = "BIC", n.models = 1,  # We only fit 1 model
# Imposing the model to use the variables we want in the best model
                     bestmodel = bestgamma,        
                     modelprior = uniform())

# Retreat coefficients information
cog.coef = coef(cog.bestbic)

out = confint(cog.coef)
names = c("post mean", "post sd", colnames(out))
coef.bic = cbind(cog.coef$postmean, cog.coef$postsd, out)
colnames(coef.bic) = names
coef.bic
```

```
##           post mean    post sd       2.5%      97.5%      beta
## Intercept 86.797235 0.87054033 85.0862025 88.5082675 86.797235
## hs         5.950117 2.21181218  1.6028370 10.2973969  5.950117
## IQ         0.563906 0.06057408  0.4448487  0.6829634  0.563906
## work       0.000000 0.00000000  0.0000000  0.0000000  0.000000
## age        0.000000 0.00000000  0.0000000  0.0000000  0.000000
```

Compared the coefficients in the best model with the ones in the full model (which can be found in the Bayesian multiple regression section), we see that the 95\% credible interval for `IQ` variable is the same. However, the credible interval for high school status `hs` slightly shifted to the right, and it is also slighly shorter, meaning a smaller posterior standard deviation. All credible intervals of coefficients exclude 0, suggesting that we have found a parsimonious model, a model that accomplishes a desired level of explanation or prediction with as few predictor variables as possible.

### Other Criteria

BIC is one of the criteria based on penalized likelihood. Other examples such as AIC (Akaike information criterion) or adjusted $R^2$, employ the form of 
$$ -2\ln(\text{likelihood}) + (p+1)k,$$
where $p$ is the number of predictors and $k$ is a constant taking different values in different criteria. BIC tends to select parsimonious models (with fewer predictor variables) while AIC and adjusted $R^2$ may include variables that are not statistically significant, but may do better for predictions.

Other Bayesian model selection decisions may be based on selecting models with the highest posterior probability. If predictions are important, we can use decision theory to help pick the model with the smallest expected prediction error. In addiciton to goodness of fit and parsimony, lost functions that include costs associated with collecting variables for predictive models may be of important consideration. 



