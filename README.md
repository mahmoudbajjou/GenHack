# ERA5-Land Bias Correction & Urban Heat Island Analysis
## GenHack : Hackathon Project

This project focuses on analyzing, visualizing, and correcting ERA5-Land meteorological data (reanalysis data) using ground station observations. Our goal was to build a machine learning pipeline capable of correcting the bias between satellite-derived temperatures and actual urban measurements in Madrid, Spain.

## Project Overview
The project is divided into four main phases, corresponding to the progression from data engineering to advanced deep learning.

## 1. Data Processing & Visualization (final-read-era5-netcdf_v2.ipynb)

- Objective: Handle high-resolution 4D NetCDF files.

- Key Tasks: * Reading ERA5-Land variables (2m temperature) using xarray.

  - Using GADM boundaries to mask global data to the specific region of Madrid.

  - Visualizing spatial temperature distributions over Europe and Spain.

## 2. Urban Heat Island (UHI) & Altitude Analysis (week2-team31-sbyyaat.ipynb)
- Objective: Understand the "Bias"—why does the satellite say one thing while the ground station says another?

- Key Tasks:

  - Calculating the discrepancy between ERA5 and ground station data.

  - Correlation analysis between Altitude, NDVI (Vegetation Index), and temperature bias.

  - Identifying that UHI effects in summer are the primary drivers of error.

## 3. Machine Learning Foundations (week3-team31-sbyyaat.ipynb)
- Objective: Predict "Reality" based on "Satellite Data."

- Models Tested:

  - Random Forest Regressor: For baseline non-linear predictions.

  - Multi-Layer Perceptron (MLP): A simple Neural Network using TensorFlow/Keras to model the bias correction.

  - Feature Engineering: Incorporating spatial coordinates and temporal features to improve accuracy.

## 4. Final Correction Pipeline (weekfinal-team31-sbyyaat.ipynb)
- Objective: A production-ready script to "clean" ERA5 data for Madrid.

- Key Results:

  - Finalized Neural Network architecture with Dropout layers to prevent overfitting.

  - Significant reduction in RMSE (Root Mean Square Error) compared to raw ERA5 data.

  - Comparison plots showing the corrected signal tracking much closer to real-world station observations.

## Technical Stack
- Data Management: xarray, pandas, numpy, netCDF4.

- Geospatial: geopandas, rioxarray, rasterio, GADM.

- Machine Learning: scikit-learn, TensorFlow, Keras.

- Visualization: matplotlib, seaborn.
