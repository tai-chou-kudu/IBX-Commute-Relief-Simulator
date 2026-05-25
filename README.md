# NYC IBX Commute Relief Estimator

**Predicting neighborhood-level commute impact of the Interborough Express, before it's built**

*Catherine Dube, Guillermo Schneider, Tai Chou-Kudu | DATA 622 Machine Learning | CUNY School of Professional Studies | Spring 2026*

---

Live App Link:  https://n6yc1z-tai0chou0kudu.shinyapps.io/IBX-Commute-Relief-Estimator/


## The Problem

Outside of Manhattan, New York's boroughs have some of the longest commute times in the country. In some outer borough neighborhoods, average commutes run as high as 53 minutes. The subway was built to move people in and out of Manhattan, not across Brooklyn and Queens. There is no fast crosstown connection between the two boroughs.

The MTA's proposed **Interborough Express (IBX)** is meant to change that. The IBX would run along an existing 14-mile freight corridor from Bay Ridge, Brooklyn to Jackson Heights, Queens, with 19 stations serving neighborhoods that the MTA describes as historically underserved by rapid transit. Close to 900,000 residents live along the proposed route, nearly three-quarters of whom are people of color.

The question this project tries to answer: what will the IBX's neighborhood-level impact look like, before the line is built?

Traditional before-and-after studies require waiting years for infrastructure to open. This project builds a what-if tool instead. We train predictive models on existing census, transit, and mobility data across all of New York City, where we already know how transit access and commute patterns relate to each other, then simulate what happens when you add transit access to the IBX corridor.

---

## What We Built

An interactive Shiny app for transit planners and policy staff who want to explore IBX impact by neighborhood without touching any R code. Users can explore:

- **IBX Simulation:** estimated drop in 60-minute-plus commute share by census tract, across six model variants
- **Demographic layers:** income, poverty rate, population density, and distance to Manhattan
- **RF Residuals map:** where the model over- and under-predicts, as a diagnostic layer
- **BART Treatment Effect map:** causal estimates of IBX impact for treated tracts only

---

## Data Sources

| Source | Description |
|--------|-------------|
| 2022 ACS 5-Year Estimates | Census-tract demographics, commute mode, income, car ownership |
| TLC Trip Records, Oct 2023 | Uber/Lyft and taxi trip counts by pickup zone |
| Citi Bike Trip Data, Oct 2023 | Station-level trip counts and station density |
| MTA IBX SEQRA Scoping Document, 2025 | Proposed IBX station addresses, geocoded to coordinates |
| MTA Subway Entrances and Exits, 2024 | Distance to nearest subway entrance per tract |

---

## Models

We built and compared six predictive model variants plus one causal model.

**Predictive models, each fit with and without spatial-lag features:**
- Linear Regression with logit-transformed outcome
- Linear + Natural Cubic Spline on subway distance
- Random Forest (500 trees, mtry=7, 10-fold CV)

**Causal model:**
- BART (Bayesian Additive Regression Trees), used to estimate the Average Treatment Effect for IBX-corridor tracts with propensity score overlap checks

### Key Finding

Model choice matters a lot in a sensitivity analysis like this. The linear baseline predicts close to zero IBX relief, not because transit access is unrelated to long commutes, but because the relationship saturates in a way that a linear model cannot represent. The spline and Random Forest models both predict around **7 to 8 percentage points of relief** for directly served tracts. The spatial-lag variants extend measurable relief to roughly 113 neighboring tracts beyond the corridor itself.

| Model | CV RMSE |
|-------|---------|
| Linear | ~8.5 |
| Linear + Spline | ~7.9 |
| Random Forest | 7.24 |
| **Random Forest + Spatial Lags** | **7.14** |

---

## Modeling Insights

**The counterintuitive transit coefficient**

The linear model finds that tracts where more workers commute by transit have *more* 60-minute-plus commutes, not fewer. This is not a causal finding. It is a geographic one. Tracts with high transit use tend to be in the outer boroughs, far from job centers, where commutes are long regardless of mode. Recognizing this kind of confounding and explaining it clearly matters as much as the model output itself.

**Diagnosing spatial structure**

Before adding `dist_to_manhattan_miles` as a feature, residuals clustered geographically, a sign the model was missing something spatial. Adding a single Manhattan proximity variable halved that clustering and improved RMSE across all three model types. The model had been using borough fixed effects to capture both borough location and Manhattan proximity at the same time. Making the two explicit and separate improved both fit and interpretability.

**Who benefits most from IBX**

The BART causal model goes beyond prediction to ask which neighborhoods would benefit most and why. Among treated tracts, the biggest estimated beneficiaries are in Queens, further from existing subway access, currently less transit-dependent, and surrounded by neighbors with higher station density. Queens treated tracts show an average treatment effect of around 0.46 percentage points of reduction in long-commute share. Brooklyn treated tracts benefit less, averaging around 0.09 percentage points. The BART magnitudes are smaller than the predictive simulation because BART applies Bayesian shrinkage toward zero and estimates a true causal effect rather than a max-relief scenario.

---

## IBX Simulation

For each of the 19 proposed IBX stations, we draw a 0.5-mile walkshed buffer around each station (about a 10-minute walk, the standard threshold for transit planning), then flag every census tract that intersects with any of those buffers as treated. The simulation sets `dist_subway_miles` to the best observed value in the dataset for treated tracts, then computes the difference between the baseline prediction and the IBX prediction.

Our best model predicts measurable relief for tracts holding close to **1 million NYC residents**, around 11% of the city's population.

---

## Limitations

The simulation overstates benefits for tracts that already have decent transit access, since it sets subway distance to the best value in the dataset for all treated tracts regardless of their starting point. The target variable only covers workers who commute to work, missing non-work trips, remote workers, and people who may have already chosen shorter-commute jobs to work around transit gaps. The model is trained on a single snapshot of data and cannot account for how patterns might shift over time. Some IBX-corridor tracts also have no good comparison group in the data, meaning the BART model has to extrapolate for them rather than match them to similar untreated tracts.

---

## Stack

R throughout. Key packages: tidycensus, sf, tigris, spdep for spatial work; arrow for the TLC parquet files; randomForest and splines for modeling; rsample for cross-validation; tidygeocoder for geocoding IBX station addresses; r5r and osmextract for multimodal routing exploration; and Shiny for the app.

---

## Setup

See the setup instructions in this repo for Git LFS and Census API key configuration, required to pull the large spatial and parquet files.

---

## Future Directions

- Validate against the Second Avenue Q extension as a natural experiment
- Full GTFS-based routing simulation for origin-destination pair estimates (r5r was explored during development; a complete routing implementation is a natural next step)
- Incorporate bus network coverage and job accessibility features
- K-fold cross-validation on the BART model
- Causal forest for heterogeneous treatment effect estimation by neighborhood
