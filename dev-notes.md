# Development Notes (preserved from original RMD)

These notes were part of the original group project development process.
Moved here during portfolio revision to keep the main document clean.

## Misc. Notes Dump
 
Spatial lag features — include averages of neighboring tract values to account for spatial autocorrelation; used in similar transit research and directly relevant to the IBX corridor analysis

K-fold cross-validation — more robust evaluation than a single 80/20 split given our dataset size

George: it might be good to have a "proximity to Manhattan" feature, otherwise the coefficients that are going to be learned for other coefficients will partially be capturing the geographic structure.

George: Second, you may want think about whether linear relationships are appropriate for all variables, particularly those which have outlier values (for example the citibike share or uber/taxi share). I also suspect the distance from subway relationship is nonlinear, like the difference from 20 minutes and 25 minutes walk to the subway might not matter. This is important because if you use a linear model, the relationship at short walk times will be constrained by the data for long walk times.

George: Try the the bartCause R package. It has built in a lot of model checks that determine how much you can truly say about causal effects given the data. read about it more here: https://apsta.shinyapps.io/thinkCausal/. IMO it is better than the CausalForest stuff from the EconML package, but that is my subjective opinion.

George: In the predicted 60m commute time, note the presence of negative values. I highly recommend applying a transformation to take the interval from 0-1 to the entire number line, i.e. a log transformation or a sigmoid function of some type. This would prevent the negative number issue.

BART mediator-as-confounder fix: An earlier version of bart_fit included `dist_subway_miles` and `lag_dist_subway_miles` in the confounders. Those are the variables the IBX actually changes. Controlling for it removed the causal pathway we were trying to measure. The original BART estimates came back with tiny positive ITE for most treated tracts (all 148 Brooklyn treated tracts positive, mean +0.5pp), which contradicted the predictive simulation. We refit BART without those two columns. The new estimates agree with the predictive simulation in direction (treatment helps most treated tracts) but with smaller magnitudes due to BART's Bayesian shrinkage toward zero.

