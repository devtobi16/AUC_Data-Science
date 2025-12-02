

````
# Selma Inclusive Growth Modeling – Mastercard AUC Data Challenge

This repository contains my work for the **Mastercard AUC Data Challenge**, focused on **Selma, Alabama**.  

The goal of the project is to:
- Understand why Selma’s **Inclusive Growth Score (IGS)** lags behind.
- Benchmark Selma against **Alabama**, the **United States**, and **peer cities**.
- Forecast how Selma’s IGS could change from **2025–2027** if we improve three key levers:
  1. **Internet Access**
  2. **Minority/Women-Owned Businesses**
  3. **Affordable Housing**

---

## 1. Project Overview

The project has three main parts:

1. **Data Exploration & Benchmarking**
   - Load and clean **Inclusive Growth Score** exports (CSV) for Selma.
   - Compare Selma’s tract-level metrics to:
     - U.S. averages (`Compared to USA.csv`)
     - Alabama averages (`Compared to State.csv`)
     - Peer cities (e.g., Detroit, Rutland, Natchez).
   - Build line charts for **2017–2024** showing Selma vs USA, Alabama, and peer cities.

2. **Policy Scenario Modeling (2025–2027)**
   - For each of the three focus areas:
     - Internet Access – benchmarked against a stronger peer city (e.g., **Detroit**).
     - Affordable Housing – benchmarked against **Rutland**.
     - Minority/Women-Owned Businesses – benchmarked against **Natchez**.
   - Use peer-city trends to project what **Selma’s tract-level values** could look like in **2025–2027** if similar programs are adopted.
   - Plot “actual vs scenario” line charts to show:
     - Selma’s historical trend (2020–2024)
     - Selma’s projected values (2025–2027) under the program scenario

3. **IGS Impact Modeling (Simple ML)**
   - Use Selma’s tract-level data to model how IGS changes with:
     - `Internet Access Tract, %`
     - `Minority/Women Owned Businesses Tract, %`
     - `Affordable Housing Tract, %`
   - Train a **linear regression** model (`scikit-learn`) on historical data.
   - Plug in the **2025–2027 projected values** for those 3 levers to estimate:
     - Selma’s **future IGS** if these programs succeed.
   - Overlay the predicted IGS on top of:
     - IGS from the **State benchmark**
     - IGS from the **USA benchmark**

---

## 2. Data

The main data files are IGS exports in CSV format, for example:

- `Selma/Compared to USA.csv`
- `Selma/Compared to State.csv`
- `Detroit/Compared to USA.csv`
- `Rutland/Compared to USA.csv`
- `Natchez/Compared to USA.csv`

Each file contains:
- **Meta rows** at the top
- A header row with metric names (e.g., `Inclusive Growth Score`, `Internet Access Tract, %`, `Female Above Poverty Tract, %`, etc.)
- One row per **census tract**, with a **Year** column

I use small helper functions (e.g. `load_igs_csv`) to:
- Skip the extra header rows
- Normalize column names
- Convert `Year` and metric columns to numeric

---

## 3. Methods & Techniques

**Tools / Libraries**
- `pandas`, `numpy` – data cleaning and aggregation
- `matplotlib` – line charts and visualizations
- `scikit-learn` – linear regression & simple modeling

**Key analysis steps**

1. **Benchmarking charts**
   - Group data by `Year` and take the **mean** of each tract-level metric.
   - Plot Selma’s average tract values vs:
     - USA “Base, %”
     - Alabama “Base, %”
     - Peer city “Tract, %”
   - Example metrics:
     - `Internet Access Tract, %`
     - `Female Above Poverty Tract, %`
     - `Minority/Women Owned Businesses Tract, %`
     - `Affordable Housing Tract, %`
     - `Residential Real Estate Value Tract, %` (explored as an infrastructure proxy)

2. **Scenario projections (2025–2027)**
   - For each peer city, compute its **annual change**:
     \[
     \Delta = \frac{\text{value}_{\text{end}} - \text{value}_{\text{start}}}{\text{years} - 1}
     \]
   - Take Selma’s **2024** value for that metric, then project 2025–2027 as:
     \[
     \text{Selma}_{2024+k} = \text{Selma}_{2024} + k \cdot \Delta
     \]
   - Plot:
     - Selma actual (2020–2024)
     - Selma projected (2025–2027) under peer-city-like improvement
     - Peer city actual (for its own window, e.g., 2019–2023)

3. **IGS prediction using 3 levers**
   - Build a modeling dataframe from Selma’s `Compared to USA.csv`:
     - Input features (X):
       - `Internet Access Tract, %`
       - `Minority/Women Owned Businesses Tract, %`
       - `Affordable Housing Tract, %`
     - Target (y):
       - `Inclusive Growth Score`
   - Drop rows with missing values, convert everything to numeric.
   - Fit a **LinearRegression** model:
     ```python
     from sklearn.linear_model import LinearRegression

     X = df_model[[
         "Internet Access Tract, %",
         "Minority/Women Owned Businesses Tract, %",
         "Affordable Housing Tract, %"
     ]].values

     y = df_model["Inclusive Growth Score"].values

     model = LinearRegression()
     model.fit(X, y)
     ```
   - Create a small `DataFrame` for **2025–2027** using the projected values from the scenario charts.
   - Use `model.predict()` to get **Predicted IGS** for each year.
   - Plot these predicted IGS points on top of the **existing IGS vs State / USA line chart**.

---

## 4. Repository Structure

An example structure (you can adjust to match your repo):

```text
.
├── data/
│   ├── Selma/
│   │   ├── Compared to USA.csv
│   │   └── Compared to State.csv
│   ├── Detroit/
│   │   └── Compared to USA.csv
│   ├── Rutland/
│   │   └── Compared to USA.csv
│   ├── Natchez/
│   │   └── Compared to USA.csv
│   └── ...
├── notebooks/
│   ├── benchmarking.ipynb
│   └── prediction.ipynb
└── README.md
````

---

## 5. How to Run

1. **Clone the repo** and create a virtual environment:

   ```bash
   git clone <your-repo-url>.git
   cd <your-repo-name>

   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

2. **Install dependencies**:

   ```bash
   pip install -r requirements.txt
   ```

   Typical dependencies include:

   * `pandas`
   * `numpy`
   * `matplotlib`
   * `scikit-learn`
   * `jupyter` / `notebook`

3. **Open the notebooks**:

   ```bash
   jupyter notebook
   ```

   Then run:

   * `01_exploration.ipynb` – basic cleaning + benchmarking charts
   * `02_scenario_projections.ipynb` – Internet, Affordable Housing, MWOB projections
   * `03_igs_prediction.ipynb` – IGS modeling + future IGS scenario plot

---

## 6. Key Takeaways (Qualitative)

* Selma lags behind U.S. and Alabama benchmarks on several **tract-level metrics**, especially:

  * Internet access
  * Women’s economic stability (female above poverty)
  * Minority/Women-owned businesses
  * Affordable housing
* Peer cities that implemented stronger programs in these areas show **clear upward trends** over a 3–4 year window.
* A simple linear regression model suggests that **improvements in these three levers** can produce a **meaningful rise in Selma’s Inclusive Growth Score by 2027**, relative to just continuing its past trajectory.

---

## 7. Limitations

* The IGS prediction model is:

  * **Linear**, and based only on **three variables**.
  * Descriptive of past relationships, not a guarantee of future outcomes.
* The projections assume:

  * Peer city trends can be transferred to Selma.
  * Other factors (labor market, health, etc.) evolve similarly to the past.

These results should be treated as **scenario estimates**, not exact forecasts.

---

## 8. Acknowledgements

* Data: Inclusive Growth Score exports provided through the **Mastercard AUC Data Challenge**.
* Tools: Python, pandas, NumPy, matplotlib, scikit-learn.

```

