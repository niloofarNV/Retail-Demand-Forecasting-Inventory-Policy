AI-Based Retail Demand Forecasting and Inventory Policy Analysis
Overview
This project explores retail demand forecasting using machine learning and analyzes how forecast errors can inform inventory decisions.

## Dataset
The project uses a selected subset of the M5 Forecasting dataset.

The original dataset is not included in this repository. Users must obtain the data from its authorized source and place the required files in the expected local directory.

## Methods
LightGBM demand forecasting
Naive and seasonal-naive baselines
Time-based training, validation, calibration, and test splits
MAE, RMSE, and WAPE evaluation
Lead-time demand uncertainty estimation
Safety stock and reorder point analysis
## Results
The LightGBM model outperformed both baseline methods on the selected test period across all three evaluation metrics.

| Model | MAE | RMSE | WAPE (%) |
|---|---:|---:|---:|
| Naive | 1.6328 | 3.5172 | 111.24 |
| Seasonal Naive | 1.2753 | 2.8183 | 86.93 |
| LightGBM | 1.1369 | 2.2499 | 77.50 |

Lower values indicate better performance across all three metrics.

The inventory analysis estimates seven-day lead-time demand, forecast-error variability, safety stock, and reorder points. The calibration procedure evaluated 49 rolling forecast origins. These estimates depend on the calibration method and modeling assumptions and do not guarantee a 95% service level.
## How to Run
Obtain the required M5 data files.
Install the dependencies listed in requirements.txt.
Place the files in the expected data directory.
Open and run the Jupyter notebook from top to bottom.
## Limitations
Safety stock and reorder point estimates depend on forecast-error calibration and modeling assumptions. Inventory service levels and cost impacts require further empirical validation.

## Author
Niloofar Vahabzadeh Najafi

## Research Interests
Supply Chain Analytics | Machine Learning | Inventory Optimization | Operations Research
