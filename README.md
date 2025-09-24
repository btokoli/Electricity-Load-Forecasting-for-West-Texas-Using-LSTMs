# Electricity Load Forecasting (ERCOT West) with LSTM

Short-term electricity demand forecasting for the **ERCOT West** zone using **LSTM** with hourly weather covariates (temperature, dew point, GHI, wind speed, relative humidity). The project compares a **region-specific model (West only)** vs. a **multi-region pretrain → West fine-tune** approach.

> **Highlight:** The West-only LSTM achieves **RMSE ≈ 28.78** vs. a naive baseline **RMSE ≈ 48.00** on 2015 test data.

---

## 📁 Repository Structure

This repo contains multiple mini-projects from the same course. The files relevant to **electricity load forecasting** are:

```
Electricity_Loading_Forecasting/
├─ 2012_ercot_hourly_load_data.xls
├─ 2013_ercot_hourly_load_data.xls
├─ 2014_ercot_hourly_load_data.xls
├─ native_load_2015.xls
├─ Texas_weather_data/               # Weather CSVs by location/source (NSRDB/NOAA/TAMU Mesonet)
├─ Data_organizing.ipynb             # Cleans/joins load + weather; windowing
├─ WeatherData.ipynb                 # Weather ETL / preprocessing
├─ TexasWeather.ipynb                # Modeling (LSTM, evaluation, plots)
├─ WEST.csv | NORTH.csv | NORTH_C.csv | FAR_WEST.csv  # Prepared region datasets
└─ (Other notebooks: HAR, GTZAN, ImageDetection2D — unrelated)
```

> 💡 The GitHub “About” text currently references KITTI/object detection from the forked template; you can ignore that—it isn’t used for this project.

---

## 🔧 Environment & Dependencies

Use Python 3.9–3.11 with Jupyter. Minimal packages:

- `pandas`, `numpy`, `scikit-learn`
- `tensorflow` (or `tensorflow-cpu`) / `keras`
- `matplotlib` (and optionally `seaborn`)

Quick setup:

```bash
# (optional) create env
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate

pip install pandas numpy scikit-learn tensorflow matplotlib seaborn jupyter
```

---

## 📊 Data

- **Load:** ERCOT hourly native load (2012–2015) for West (and other regions for pretraining).
- **Weather:** Hourly covariates for the same period/region from NSRDB/NREL and NOAA/TAMU Mesonet.
- **Alignment:** Datasets merged by timestamp (`Year, Month, Day, Hour`).
- **Cleaning:** Linear interpolation to address gaps/outliers; features normalized; target (`electric load`) kept in native units.
- **Windowing:** 24-hour look-back to predict the next hour (t+1).

---

## ▶️ How to Reproduce

1. **Open & run** `WeatherData.ipynb`  
   - Consolidates weather sources and exports cleaned hourly weather tables.

2. **Open & run** `Data_organizing.ipynb`  
   - Merges load + weather, handles interpolation/normalization, and creates 24-hour sliding windows.  
   - Exports per-region datasets (e.g., `WEST.csv`).

3. **Open & run** `TexasWeather.ipynb`  
   - Trains/evaluates LSTM for two setups:
     - **Approach A (West-only):** Train/val on 2012–2014 (80/20 split), test on 2015.
     - **Approach B (Multi-region):** Pretrain on North / North Central / Far West, then fine-tune on West, test on 2015.
   - Includes early stopping and evaluation plots (Loss/RMSE, actual vs. predicted).

> Tip: If you want deterministic runs, set all relevant RNG seeds in NumPy/TensorFlow and configure a deterministic backend.

---

## 🧠 Model

- **Architecture:** LSTM layer → Dense layers (ReLU) → Linear output for 1-step ahead regression.
- **Training:** Early stopping (max ~30 epochs), MSE loss, Adam optimizer.
- **Features:** 24-hour lag window of load + weather; target is next-hour load (West).

---

## ✅ Results (2015 Test)

| Setup                                   | Test MSE | Test RMSE | Notes                                  |
|-----------------------------------------|---------:|----------:|----------------------------------------|
| **Approach A — West-only LSTM**         | ~828.51  | **28.78** | Tracks peaks/valleys; strong baseline |
| **Naive baseline (persistence)**        |    —     | **48.00** | For reference                         |
| **Approach B — Multi-region → West FT** | ~8814.98 | **93.89** | Underfits West after pretraining      |

Interpretation: Region-specific training on West data generalizes best here; naive multi-region pretraining without careful adaptation can over-smooth variability. Consider domain-aware pretraining (e.g., covariate shift handling, curriculum, or adapter layers) before fine-tuning.

---

## 📈 Plots (in notebooks)

- Train/validation **Loss (MSE)** and **RMSE** curves  
- **Actual vs. Predicted** (2015 West)

---

## 🗺️ Roadmap / Next Steps

- Add **holiday/calendar** and **market** signals (e.g., price, outages) as exogenous features  
- Try **multi-step** forecasting (seq2seq or direct multi-horizon)  
- Hyperparameter search (units, layers, dropout, look-back sizes)  
- **Explainability:** permutation importance on covariates; SHAP for sequence models  
- Explore **transfer learning** with domain adaptation (e.g., CORAL, adversarial alignment)

---

## 📚 References & Data Sources

- ERCOT Native Load (hourly)  
- NREL **NSRDB** (solar/irradiance & weather)  
- NOAA/NCEI, NWS, and Texas A&M Mesonet (supplementary weather)

Suggested academic references:
- Hossain & Mahmood (2020), “Short-Term Load Forecasting Using an LSTM Neural Network,” IEEE PECI.  
- Hossain & Mahmood (2020), “Short-Term Photovoltaic Power Forecasting Using an LSTM Neural Network and Synthetic Weather Forecast,” IEEE Access.

> See the project report in this repo/assignment for full citations and methodology details.

---

## ⚖️ License

No explicit license provided. If you plan to open-source, consider adding an MIT/Apache-2.0 license and include attribution requirements for data sources.

---

## 🙏 Acknowledgments

Course project by **Bismack Tokoli** and collaborators. Weather and load data courtesy of their respective providers (NREL/NSRDB, NOAA/NWS/NCEI, Texas A&M Mesonet, ERCOT).
