---
title: "On questioning Roth C model for Agreena"
description: "On questioning Roth C model for Agreena"
tags: Agreena
---


- Who maintains and updates the Roth C implementation used in the model? Do you have technical capacity and resources to properly use the model?

Primarily Roth C is implemented by Marcos from Climate Impact Research. Research Software Engineer for Github maintainence, creating timeline, needs to file exchange. 3rd Modeller working partially doing PHD in company. And Junior data analyst.

- Would you describe your approach – “Measure and model“ + “default factors“ from IPCC?
Measure and model approach. Soil samples are used to initialize 5 carbon pools initialize pool using optimization process. Initialize -5 years to t0 run, two more run - baseline run (10 years) and scenario run (10 years) - but change in management. They compare and deduct. 

- The project area is over several countries. How is the modelling approach suitable for the different local conditions? You mention variations in crop combinations and practices. But how do you account for variation in local conditions?
Part of this variation is done by data collection variation. High res soil grids. Calibration coefficients for 4 major climate zones in Europe.


- Were these datasets region-specific (e.g., for Hungary), or more general?
They are even local specific more granual region-specific. Runs happen for fields seperately.

- What sources of data were used to calibrate the Roth C parameters?
Primarily scientific litreture and papers. No private dataset. Info for each individual field data for mangement field.


- For the intial SOC values, how is the sample size determined for soil sampling? How is the region stratified?
Environmental covariates and political covariets. Climate and crop type variables using k-means clustering averaging. Avg shared by fields within the cluster.

- IDD Table A.4.1: "SOC and BD values to initiate the models ( SOCinit) are determined using gridded values acquired from publicly available libraries (SoilGrids).Measured values will be used for the model true-up."


IDD Section 4.5: "In compliance with the requirements, SOC stock changes are modelled using measured SOC as the initial SOC in the RothC simulations. For ex-ante evaluations, SoilGrids SOC stock values at the local level are used, available at a resolution of 250 m². For ex-post calculations, SOC measured according to the stratification strategy outlined in Section 4.5.7 are mapped to individual fields and used for model initialization."

The two statements contradict.

A. SoilGrid is used to get Clay content and soil texture. Initial SOC was needed so they got it from SoilGrid. Based on soil samples SOCInt would be updated as more and more soil data is used. SC - use SoilGrid to initiate the model and then use Soil samples for updating the model for us. In VERRA the contract is longer and 1 year in SC.


- How do you ensure that the approach remains conservative and sufficiently accurate?
Through the calibration estimated errors associated with the model that they used to deduct the model results.


- Ogle et al. (2019) underscore that net SOC gains under tillage are rare, inconsistent, and often linked to measurement artifacts (e.g., changes in bulk density or sampling depth not properly corrected). EMS literature expresses the same thing. How is the impact of bulk density change reflected in roth-C model?


- What statistical metrics were used to evaluate the model fit (e.g., RMSE, MAE, R²)? Bias estimation is computed. Statistical measurments are performed along with RMSE.
- Can you provide calibration error ranges?




- How is tillage intensity defined and modeled in the roth C model? How do you define "reduced tillage"?
As a tillage rate modifier. Based on a european study from 2022, calibration factors where taken from it. 3 factors - no tillage, full tillage, reduced tillage.

- How are the studies for model validation selected?




Section 2.12 describes optimisation of three calibration factors (cf1cf3) using bounded global optimisation (NLopt) with MSE as the objective and group kfold crossvalidation to select validation statistics.

- Reference to Section 2 but missing.
Its from Model Asessment report.


- Calibration dataset - not full but partial that shows variability.
- What is the source of Validation Results in Section 3.4. of the Model Assessment Report? After calibrating the model and the results from the refences and summarizing the results of model vs measurements following the equeations from VM0053. 

- How is cross-validation done?
k14 - If no. of datapoint is smaller than 5 for a specific point then point is left out. The one left out is validated seperately. The others are validated. They are then summed up and averaged to calcuate RMSE. 

- How do you differentiate dataset for Validation and Calibration?
All the error measurement was calculated when the model was not. Error measurement is independent.  


- Is there is any scope change between VERRA and SC? In running Roth C there are no differences.

- Did you evaluate model against observed SOC stock changes in field trials? 

- How are field-level inputs (e.g., tillage, residue management, fertilizer type) collected and verified?
Collected at field level. 


- Are these inputs farmer-reported or system-generated?
Two way validation process and validated through remote sensing or vice versa. 

- What databases or rulesets govern default values for missing parameters (e.g., carbon content of mulch, fertilizer emissions)?
Box 1 from VM0042 is used. Farm attestations are used. Box 4 lowest priority is regional data. Project is split into different geographies - and data and parameter is selected for each geography. Is there missing data, data imputation method - using weighted averages for missing data.

- How is residue handeled in the model?
Reported values from farmers. The residues reported, the equations from IPCC2019 to convert crop yield to crop residues.  

- Is there an upper bound to modeled SOC accumulation over time?
No. 

- How sensitive is the model output to uncertainties in rainfall, 
temperature, or soil texture inputs?

- Are there automated checks for inconsistent field-level entries (e.g., organic fertilizer with zero N)?
Yes, Ben can cover it. Its a QA QC issue. Min max threshold are there that filters values.

- Can you trace the output SOC delta back to specific field inputs and Roth C parameter sets?
Its possible to trace the output to inputs but a specific activity like tillage or mulch.

- If a verifier reruns the model with the same inputs, will they get the same result?
Definately. 