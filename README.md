# Hydroponics IoT & Machine Learning
This repository contains the final project for AAI-530: Data Analytics and IoT, focused on applying machine learning and time series analysis to IoT sensor data from an autonomous hydroponics system.

The project explores how continuous water quality monitoring data can be used to generate predictive insights and inform autonomous strategies in controlled-environment agriculture.

## Project Overview

Hydroponic systems rely on precise control of water quality parameters to maintain plant health and productivity. Because these systems operate continuously and are sensitive to small environmental changes, reliable monitoring and analytical insight are critical for stable operation.

This project uses real IoT sensor data from an experimental hydroponics platform to explore how machine learning can support monitoring, prediction, and performance analysis in automated growing systems.

Specifically, the project aims to:
- Design a complete theoretical IoT system architecture
- Perform exploratory data analysis and preprocessing on raw sensor data
- Apply traditional machine learning and deep learning methods to multivariate time series data
- Generate predictive insights related to water quality behavior
- Communicate results through clear visualizations and a dashboard

## File Structure
`.`<br>
`├── data/                # Data files`<br>
`├── notebooks/           # Exploratory analysis and model development notebooks`<br>
`├── models/              # Machine learning and deep learning implementations`<br>
`├── visualizations/      # Generated plots and figures`<br>
`├── README.md`<br>
`└── LICENSE`

## Environment Setup

Primary environment management uses Conda (`environment.yml`).

A `requirements.txt` file is also provided for pip-based setups such as Google Colab or quick local testing.

### Setup Instructions
Clone the repository
```
git clone https://github.com/seanmcgowanx/iot-timeseries-hydroponics.git
cd iot-timeseries-hydroponics
```
Create the Conda environment
```
conda env create -f environment.yml
```
Activate the environment
```
conda activate hydroponics-iot
```
For pip-only environments such as Google Colab:
```
pip install -r requirements.txt
```
## Authors
- Sean McGowan
- Thiago Alvares
- Jacob Edwards
