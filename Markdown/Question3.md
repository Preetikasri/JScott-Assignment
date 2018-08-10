---
title: "Hw3"
Group: 11(Preetika Srivastava, Vir Mehta, Cherry Agarwal, Guanhua Zhang) 
date: "2018/8/7"
output: 
 html_document:
    keep_md: true
---

```{r}
rm(list=ls())
#install.packages('quantmod')
library(quantmod)
library(mosaic)

library(foreach)
sp500 <- new.env()
SPY=getSymbols("SPY",auto.assign = FALSE, from = "2005-01-01",to='2018-01-01')
TLT=getSymbols("TLT",auto.assign = FALSE, from = "2005-01-01",to='2018-01-01')
LQD=getSymbols("LQD",auto.assign = FALSE, from = "2005-01-01",to='2018-01-01')
EEM=getSymbols("EEM",auto.assign = FALSE, from = "2005-01-01",to='2018-01-01')
VNQ=getSymbols("VNQ",auto.assign = FALSE, from = "2005-01-01",to='2018-01-01')


TLTa = adjustOHLC(TLT)
LQDa = adjustOHLC(LQD)
EEMa = adjustOHLC(EEM)
VNQa = adjustOHLC(VNQ)
SPYa = adjustOHLC(SPY)

hist(ClCl(SPYa),5, main = "Distribution for SPY returns ")
hist(ClCl(TLTa),5, main = "Distribution for TLT returns ")
hist(ClCl(LQDa),5, main = "Distribution for LQD returns ")
hist(ClCl(EEMa),5, main = "Distribution for EEM returns ")
hist(ClCl(VNQa),5, main = "Distribution for VNQ returns ")

all_returns = cbind(ClCl(SPYa),ClCl(TLTa),ClCl(LQDa),ClCl(EEMa),ClCl(VNQa))
stdev_vector = vector()
for (i in 1:ncol(all_returns))
{
 s = sd(all_returns[, i], na.rm = T)
 stdev_vector = c(stdev_vector, s)
 
}

all_etf = colnames(all_returns)
ETF_order = all_etf[order(stdev_vector, decreasing = F)]
stdev_vector = stdev_vector[order(stdev_vector, decreasing = FALSE)]
print(ETF_order)
## The last element is the 
# most volatile one

all_returns_1 = cbind(ClCl(SPYa),ClCl(TLTa),ClCl(LQDa),ClCl(EEMa),ClCl(VNQa))
head(all_returns_1)
all_returns_1 = as.matrix(na.omit(all_returns_1))

return.today_1 = resample(all_returns_1, 1, orig.ids=FALSE)

initial_wealth = 100000
sim1 = foreach(i=1:1000, .combine='rbind') %do% {
	total_wealth_1 = initial_wealth
	weights_1 = c(0.2, 0.2, 0.2, 0.2, 0.2)
	
	n_days_1 = 20
	wealthtracker_1 = rep(0, n_days_1)
	for(today in 1:n_days_1) {
	  holdings_1 = weights_1 * total_wealth_1
		return.today_1 = resample(all_returns_1, 1, orig.ids=FALSE)
		holdings_1 = holdings_1 + holdings_1*return.today_1
		total_wealth_1 = sum(holdings_1)
		wealthtracker_1[today] = total_wealth_1
	}
	wealthtracker_1
}

head(sim1)
hist(sim1[,n_days_1], 25, main = "Distribution of Wealth")

# Profit/loss
mean(sim1[,n_days_1])
hist(sim1[,n_days_1]- initial_wealth, breaks=30, main = "Distribution of Loss/Profit")

# Calculate 5% value at risk
q = quantile(sim1[,n_days_1], 0.05) - initial_wealth


print("We have 5% chance that loss will be more than :")
print(q)

```

The histograms of each asset show the distribution for the daily return of each assets. The more volatile the return is, the more risk the investor will bear. Also, the higher return also involves more risk of loss. Here we calculate the variance of each asset’s return and find the more risky assets: VNQ and EEM, and the safer assets: LQD, TLT, SPY.



```{r}
### Simulating weights for ETFs to distribute the wealth

set.seed(99)
weights.mat = matrix(0.00, nrow = 50, ncol = 5 )
for (k in (1:nrow(weights.mat)))
{
 #print(k)
 voltl = runif(2, 0.05, 0.1)
 weights.mat[k,4:5] = voltl
 safe = runif(2,0.2,0.3)
 last = 1- sum(c(voltl, safe))
 v1 = c(safe, last)
 v1 = v1[sample(1:3)]
 weights.mat[k, 1:3] = v1
}

## Using these weights to simulate different wealth distributions. However, for one single distribution weights remain ##fixed.

all_returns_2 = cbind(ClCl(SPYa),ClCl(TLTa),ClCl(LQDa),ClCl(EEMa), ClCl(VNQa))
head(all_returns_2)
all_returns_2 = as.matrix(na.omit(all_returns_2))

return.today_2 = resample(all_returns_2, 1, orig.ids=FALSE)

initial_wealth = 100000

comparison.mat = matrix(0, nrow = 50, ncol = 2)
for (r in 1:nrow(weights.mat))
{
  sim2 = foreach(i=1:1000, .combine='rbind') %do% {
  total_wealth_2 = initial_wealth
  weights_2 = weights.mat[r, ]
  n_days_2 = 20
  wealthtracker_2 = rep(0, n_days_2)
  
  for(today in 1:n_days_2) {
   holdings_2 = weights_2 * total_wealth_2
   return.today_2 = resample(all_returns_2, 1, orig.ids=FALSE)
   holdings_2 = holdings_2 + holdings_2*return.today_2
   total_wealth_2 = sum(holdings_2)
   wealthtracker_2[today] = total_wealth_2
  }
 wealthtracker_2
}

# Profit/loss
mean(sim2[,n_days_2])
#hist(sim2[,n_days_2]- initial_wealth, breaks=30)

# Calculate 5% value at risk
q= quantile(sim2[,n_days_2], 0.05) - initial_wealth
comparison.mat[r, 1] = r
comparison.mat[r,2] = q

}

idx_max = which(comparison.mat[,2] == max(comparison.mat[, 2]))
#optimum_weights = weights.mat[idx_max, ]

optimum_weights = data.frame(matrix(ncol = 5, nrow = 1))
names(optimum_weights) = c("SPY", "TLT", "LQD", "EEM", "VNQ")
optimum_weights[1,] = weights.mat[idx_max, ]

print(" The optimum distribution of wealth by this simulation is calculated as: ")
optimum_weights


```

We decided to choose all the five ETFs, and assign simulated weights to them. We inspected the standard deviation for each of them and found which ones are prone to heavy fluctuations. The most aggressive ones are assigned very low weights, compared to the safer ones. And we keep simulating the weights also. 

The ditribtuion we get can be seen in the results , and how the wealth is distributed across the assets. 
The first three which are safe are having maximum wealth distribution 

It is quite evident how the best set of weights are distributing the wealth optimally across all assets.





```{r}


### part 3

## Simulating weights for two most aggresive assets

set.seed(99)
weights.mat_aggresive = matrix(0.00, nrow = 50, ncol = 2 )
for (k in (1:50))
{
 #print(k)
 weights.mat_aggresive[k,1] = runif(1,0.0, 1.0)
 weights.mat_aggresive[k,2] = 1- (weights.mat_aggresive[k,1])
}

## Choosing the most aggressive two assetts. 
all_returns_3 = cbind(ClCl(EEMa), ClCl(VNQa))
all_returns_3 = as.matrix(na.omit(all_returns_3))

return.today_3 = resample(all_returns_3, 1, orig.ids=FALSE)

initial_wealth = 100000

comparison.mat.agg = matrix(0, nrow = 50, ncol = 2)
for (r in 1:nrow(weights.mat_aggresive))
{
 sim3 = foreach(i=1:1000, .combine='rbind') %do% {
   total_wealth_3 = initial_wealth
   weights_3 = weights.mat_aggresive[r, ]
   #holdings_3 = weights_3 * total_wealth_3
   n_days_3 = 20
   wealthtracker_3 = rep(0, n_days_3)
   for(today in 1:n_days_3) {
     
     holdings_3 = weights_3 * total_wealth_3
     return.today_3 = resample(all_returns_3, 1, orig.ids=FALSE)
     holdings_3 = holdings_3 + holdings_3*return.today_3
     total_wealth_3 = sum(holdings_3)
     wealthtracker_3[today] = total_wealth_3
   }
   wealthtracker_3
 }
 
 # Profit/loss
 mean(sim3[,n_days_3])
 
 
 # Calculate 5% value at risk
 q= quantile(sim3[,n_days_3], 0.05) - initial_wealth
 comparison.mat.agg[r, 1] = r
 comparison.mat.agg[r,2] = q
 
}


idx_max = which(comparison.mat.agg[,2] == max(comparison.mat.agg[, 2]))
#optimum_weights = weights.mat[idx_max, ]

optimum_weights = data.frame(matrix(ncol = 2, nrow = 1))
names(optimum_weights) = c( "EEM", "VNQ")
optimum_weights[1,] = weights.mat_aggresive[idx_max, ]

print(" The optimum distribution of wealth by this simulation is calculated as: ")
optimum_weights



```


In this case, we decided to choose VNQ and EEM in our aggressive portfolio. We decided to not go with all the assets , since the risk of portfolio will be diversified with more assets included and it will get safer. Here, we are looking out for the most risky portfolios.We have tried 50 weights combinations for each portfolio and find the best one and worst one and make our investment recommendation based on our result.

Based on the results here, we see that the best wealth distribution 

Based on the results of three portfolios, we see that how assets can be distributed btw two ETFs. 

