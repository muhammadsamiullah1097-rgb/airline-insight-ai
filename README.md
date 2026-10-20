# ✈️ AirlineInsight AI

A small **full-stack machine learning web app** that solves two real airline problems:

1. **Arrival Delay Predictor** — predicts how late a flight will arrive (a *number*).
2. **Passenger Satisfaction Analyzer** — predicts whether a passenger will be satisfied (a *category*).

Built with **scikit-learn**, **FastAPI** and a single **HTML/CSS/JavaScript** page.

---

## 📊 Dataset

[Airline Passenger Satisfaction — Kaggle](https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction)

- 129,880 passenger records (103,904 train + 25,976 test, the split given with the dataset)
- Each record has passenger details, flight distance, delays and 14 service ratings (0–5), plus the final `satisfaction` label.

> The CSV files are **not** included in this repo (they are large). Download `train.csv` and `test.csv` from the Kaggle link above and place them in the `data/` folder.

---

## 🤖 The Two Models

| # | Task | Model | Input | Output | Score |
|---|------|-------|-------|--------|-------|
| 1 | Regression | **Linear Regression** | Departure Delay, Flight Distance | Arrival Delay (minutes) | **R² = 0.9302**, MAE = 5.27 min |
| 2 | Classification | **Logistic Regression** | 14 service ratings, Age, Flight Distance, Gender, Customer Type, Type of Travel, Class | `satisfied` / `neutral or dissatisfied` | **Accuracy = 87.04%**, ROC-AUC = 0.9248 |

Both models are trained in `train_models.py` and saved with **joblib** into the `models/` folder.

**In simple words:**
- The regression model learned this formula: `Arrival Delay ≈ 0.92 + 0.98 × Departure Delay` — a flight that leaves late arrives almost equally late.
- The classification model first prepares the data (numbers are scaled with `StandardScaler`, text columns are converted with `OneHotEncoder`) and then Logistic Regression predicts the category, together with a confidence probability.

---

## 📁 Project Structure

```
airline-insight-ai/
├── train_models.py          # Trains both models, prints R2 + accuracy, saves them with joblib
├── requirements.txt         # Python libraries needed
├── data/                    # Put train.csv and test.csv here (from Kaggle)
├── models/
│   ├── regression_model.pkl       # Saved Linear Regression model
│   ├── classification_model.pkl   # Saved Logistic Regression model
│   └── model_info.json            # Model scores, shown on the website
└── app/
    ├── main.py              # FastAPI backend (loads models, 2 POST endpoints)
    └── static/
        └── index.html       # Frontend: one page, one form per model
```

---

## 🚀 How to Run

### 1. Install the libraries
```bash
pip install -r requirements.txt
```

### 2. Train the models (first time / fresh clone)
Put `train.csv` and `test.csv` in `data/` (see the Dataset section above), then:
```bash
python train_models.py
```
This prints the R² / accuracy scores and saves the models with joblib.
*(If you received this project as a zip, the trained `.pkl` models are already in `models/` and you can skip this step and go straight to Step 3.)*

### 3. Start the backend (FastAPI)
```bash
uvicorn app.main:app --reload
```

### 4. Open the website
Go to **http://127.0.0.1:8000** in your browser.

- The left/top form predicts **arrival delay**.
- The right/bottom form predicts **passenger satisfaction**.
- Use the preset buttons (e.g. *“Happy passenger”*) for a one-click demo.
- Interactive API docs are also available at **http://127.0.0.1:8000/docs**

---

## 🔌 API Endpoints

### `POST /predict/regression`
Request:
```json
{ "departure_delay": 30, "flight_distance": 1200 }
```
Response:
```json
{ "predicted_arrival_delay_minutes": 30.1 }
```

### `POST /predict/classification`
Request (short example — all 14 ratings are required):
```json
{
  "age": 35, "flight_distance": 1200, "gender": "Female",
  "customer_type": "Loyal Customer", "travel_type": "Business travel",
  "flight_class": "Business",
  "inflight_wifi": 5, "time_convenient": 4, "online_booking": 5,
  "gate_location": 4, "food_and_drink": 5, "online_boarding": 5,
  "seat_comfort": 5, "inflight_entertainment": 5, "onboard_service": 5,
  "leg_room": 4, "baggage_handling": 5, "checkin_service": 5,
  "inflight_service": 5, "cleanliness": 5
}
```
Response:
```json
{ "prediction": "satisfied", "probability_satisfied": 0.9914 }
```

### `GET /model-info`
Returns the model scores (R², MAE, accuracy, ROC-AUC) shown in the website header.

---

## ⚠️ Limitations (honest notes)

- The delay model mostly uses the **departure delay**, so it is an *estimate at/after departure*, not a guaranteed forecast days before the flight.
- Satisfaction is predicted from the passenger's **own survey ratings**, so the classifier is best used to understand which services drive satisfaction.

## 🛠️ Tech Used
Python · pandas · scikit-learn · joblib · FastAPI · HTML/CSS/JavaScript
