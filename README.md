# Smart Meter Load Forecasting and Anomaly Detection

A reproducible portfolio project using the [UCI Electricity Load Diagrams 2011–2014 dataset](https://archive.ics.uci.edu/dataset/321/electricityloaddiagrams20112014). It explores consumption patterns for 370 clients, forecasts the next 24 hours at 15-minute resolution, and flags unusual readings using forecast errors.

## Project contents

- `smart_meter_load_forecasting.ipynb` — data checks, exploratory analysis, forecasting, anomaly flags, and evaluation.
- `requirements.txt` — Python dependencies.
- `data/` — local dataset location (ignored by Git).

## Dataset setup

Download `LD2011_2014.txt` from the [UCI dataset page](https://archive.ics.uci.edu/dataset/321/electricityloaddiagrams20112014) and place it at `data/LD2011_2014.txt`. The notebook reads the semicolon-delimited file directly and treats `?` as missing. The source file is large (about 678 MB uncompressed); it is intentionally not included in this repository.

The dataset is licensed under CC BY 4.0. Cite the source as: Trindade, A. (2015). *ElectricityLoadDiagrams20112014*. UCI Machine Learning Repository. https://doi.org/10.24432/C58C86.

## Run

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter notebook smart_meter_load_forecasting.ipynb
```

Run the notebook from top to bottom. It parses and sorts timestamps, averages duplicate timestamps, creates a 15-minute grid, and interpolates only interior gaps of up to one hour. Longer gaps remain missing and are excluded from model fitting and scoring. The last 20% of timestamps are held out chronologically; an earlier validation interval calibrates per-client anomaly thresholds.

## Modeling and interpretation

The seasonal-naive baseline repeats the previous day at the same quarter-hour. A pooled Random Forest predicts one step ahead from recent lags, rolling consumption summaries, cyclic time features, and one-hot encoded client identity; it recursively generates a 96-step day-ahead path. The fit uses at most 500,000 temporally distributed training rows. Forecast quality is reported with MAE and RMSE across weekly forecast origins, both pooled and per client.

Anomalies are readings whose one-step absolute forecast error exceeds that client’s 99th-percentile validation error. These are statistical flags for review, not confirmed equipment faults; the dataset does not include anomaly labels.

## Notes

- The dataset values are consumption in kW, as described by UCI.
- Missing values and clients with insufficient recent history can reduce the number of observations available for forecasts and metrics; the notebook reports coverage.
- Results depend on the selected random training sample and the dataset’s historical period. The notebook fixes its random seed for reproducibility.
