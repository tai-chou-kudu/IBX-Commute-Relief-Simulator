# Modeling the Impact of the Interborough Express on NYC Commute Times

**Tai Chou-Kudu** | DATA 622, Spring 2026
Originally developed with Catherine Dube and Guillermo Schneider.

## What This Project Does

This project estimates how the proposed [Interborough Express (IBX)](https://new.mta.info/project/interborough-express) subway line would change commute times for the Brooklyn and Queens neighborhoods it would serve. We combine Census demographics, TLC taxi/rideshare data, Citi Bike trip data, and MTA subway station locations to build predictive and causal models of long commutes (60+ minutes) at the census tract level across NYC.

The core question: if you drop a new subway line into underserved parts of Brooklyn and Queens, how much does it actually move the needle on commute times?


## My Contributions

This was a three-person group project. Here is what I built:

**Random Forest model pipeline** -- This was the main modeling contribution I made. I built the full RF pipeline from scratch on a feature branch (`tai/random-forest-model`), including training and evaluation, the IBX simulation with RF predictions, integration of borough fixed effects, a residuals map layer for the Shiny app, clamping so predicted percentages stay in [0, 100], and a fix for a GEOID factor-level bug that was breaking both the RF and linear models.

**Shiny app visualization** -- I built the RF-specific map layers in the Shiny app: viridis colorscale (colorblind-friendly), separate palettes for predictions vs. relief vs. residuals (Blues / viridis / RdBu), and iterated on those until they were easy to read.

**Feature engineering** -- I added `pct_2plus_cars` from ACS table B25044 as a car-ownership predictor, and restored a vehicle availability chunk that got lost in a merge conflict.

**Portfolio revisions** -- After receiving professor feedback, I created this portfolio branch and made the following improvements:
- Fixed a data bug in `pct_no_car` (was dividing household counts by population instead of by total households)
- Added an explicit treatment definition for the BART causal section
- Added IBX site-specific treatment effect reporting
- Added a paragraph explaining how spatial lags are constructed (queen contiguity, row-standardized weights)
- Added partial dependence plots for the top RF features to show direction of effect
- Fixed propensity score diagnostics to exclude the treatment mechanism (`dist_subway_miles`)
- Switched BART ITE maps from a diverging palette to a sequential one
- Added quantile capping for outlier-heavy color scales (uber trips, bike access)
- General cleanup of typos and internal dev notes

**What my teammates built:** Catherine built the initial data pipeline, linear and spline models, distance-to-Manhattan feature, 10-fold CV framework, spatial lag features, and most of the Shiny app infrastructure. Guillermo built the BART/bartCause causal inference section.

## Report
Full write-up: [NYC IBX Commute Relief Estimator (PDF)](NYC%20IBX%20Commute%20Relief%20Estimator.pdf)

*Note: this report reflects the full group submission by Catherine Dube, Guillermo Schneider, and Tai Chou-Kudu. My individual contributions are described in the My Contributions section above.*


## Models

We trained six predictive model variants (Linear, Linear + Spline, Random Forest, each 
with and without spatial lag features) and one causal model (BART via `bartCause`). 
10-fold cross-validation picked Random Forest + Spatial Lags as the best performer.

One finding worth noting: after correcting the `pct_no_car` denominator bug (see My 
Contributions above), the linear model produces zero predicted IBX relief. This is the 
correct result -- `dist_subway_miles` was never significant in the fully-specified linear 
model (p=0.62), and the data fix removed the artificial variance that had been masking 
this. The spline and RF models are robust to this because they capture the nonlinear 
distance-to-subway relationship the linear model cannot. This is one reason RF + spatial 
lags was the right model choice for the IBX simulation.

The IBX simulation sets each treated tract's distance-to-subway to the city-wide minimum 
z-score and re-predicts commute outcomes. Spatial lag variants also recompute the 
neighbor-averaged distance feature so spillover effects propagate to adjacent tracts.

## How to Run

### Prerequisites
- R 4.x with RStudio or Positron
- Git LFS (the repo has large `.parquet` and `.rds` files)
- A free Census API key from https://api.census.gov/data/key_signup.html

### Setup

```bash
git lfs install
git lfs pull
```

In R:
```r
usethis::edit_r_environ()
# Add: CENSUS_API_KEY=your_key_here
# Restart R
```

### Run
- Knit `DATA 622 - Project MVP V1.Rmd` to regenerate all models and outputs
- Run `app.R` for the interactive Shiny map

## Data Sources
- **Census:** 2022 ACS 5-Year Estimates (demographics, commute modes, income, housing)
- **TLC:** October 2023 yellow taxi, green taxi, and Uber/Lyft trip records
- **Citi Bike:** October 2023 trip data
- **MTA:** Subway Entrances and Exits 2024, IBX station locations from the SEQRA scoping document

## Project Structure

```
DATA 622 - Project MVP V1.Rmd   # Main analysis (knit to reproduce everything)
app.R                            # Shiny app for interactive map
data/                            # Saved RDS model outputs (generated by knit)
citibike/                        # October 2023 Citi Bike trip CSVs
taxi_zones/                      # TLC taxi zone shapefiles
dev-notes.md                     # Archived development notes from group project
```
