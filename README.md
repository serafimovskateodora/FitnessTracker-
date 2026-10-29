# Fitness Tracker: Sensor Data Analysis for Gym Exercise Recognition

A data-processing pipeline for wearable sensor data (accelerometer and gyroscope) recorded while participants perform gym exercises. The goal is to turn raw, noisy, multi-rate sensor streams into a clean, unified dataset that is ready for machine learning-based exercise recognition.

> **Note:** Personal learning project, built while following an online course on sensor-based exercise recognition. [Add course name / link here.]

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Pipeline](#pipeline)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Results and Figures](#results-and-figures)
- [Key Learnings](#key-learnings)
- [Next Steps](#next-steps)
- [Acknowledgements](#acknowledgements)

---

## Overview

Wearable sensors produce large volumes of time-series data, but raw sensor data cannot be used directly for machine learning. Files are split per sensor, the sensors record at different frequencies, and the signals contain spikes and measurement errors.

This project handles that preparation work:

1. Loads and merges many raw CSV files into one dataset.
2. Aligns two sensors that record at different sampling rates.
3. Detects and removes outliers using three different methods.
4. Visualizes the movement patterns of each exercise and participant.

## Dataset

Data was recorded with a **MetaMotion** wearable sensor, which captures:

| Sensor        | Sampling rate | Signals          |
|---------------|---------------|------------------|
| Accelerometer | 12.5 Hz       | `acc_x/y/z`      |
| Gyroscope     | 25 Hz         | `gyr_x/y/z`      |

Participants performed gym exercises such as **bench press, squat and row** in **medium** and **heavy** sets. Metadata (participant, exercise, set category) is parsed from each file name.

[Add: number of participants, number of exercises, and a link to the data source if it is public.]

## Pipeline

### 1. Data loading and merging

- Reads all raw CSV files from the sensor folder.
- Extracts **participant, exercise label and category** from each file name.
- Assigns a unique **set number** to every recording.
- Converts the epoch timestamp to a `DatetimeIndex` and drops redundant time columns.

### 2. Merging and resampling

- Joins accelerometer and gyroscope data into a single table.
- Resamples to **200 ms** intervals using the mean of numeric signals, and the last value for metadata columns.
- Resamples **day by day** so that gaps between recording sessions are not filled with empty rows.

**Why:** the two sensors run at different frequencies (12.5 Hz vs 25 Hz), so their timestamps do not line up. A common time grid is required before any model can use both signals together.

### 3. Outlier detection

Three methods were compared:

| Method | Type | Idea |
|--------|------|------|
| **IQR** | Distribution-based | Flags values outside 1.5 x IQR from the quartiles |
| **Chauvenet's criterion** | Distribution-based | Flags values whose probability under a normal distribution is below a threshold |
| **Local Outlier Factor (LOF)** | Distance-based | Flags points whose local density is much lower than their neighbors' |

**Chosen method: Chauvenet's criterion, applied per exercise.** It assumes a normal distribution, and each exercise has its own signal distribution, so fitting it per exercise avoids flagging normal movements of one exercise as outliers just because they look unusual compared to another. Detected outliers are replaced with `NaN` rather than deleted, so they can be handled in a later step.

### 4. Visualization

- Single-axis plots per set.
- Comparison of **medium vs heavy** sets for the same exercise.
- Comparison of **participants** for the same exercise.
- Combined accelerometer and gyroscope plots for every exercise/participant combination, exported as PNG files.

## Tech Stack

- **Language:** Python
- **Data processing:** pandas, NumPy, SciPy
- **Machine learning utilities:** scikit-learn (Local Outlier Factor)
- **Visualization:** matplotlib
- **Tools:** PyCharm, Git

## Project Structure

```
FitnessTracker/
├── data/
│   ├── raw/MetaMotion/            # raw sensor CSV files
│   └── interim/                   # processed datasets (.pkl / .csv)
├── docs/
├── models/                        # saved models
├── notebooks/
├── references/
├── reports/
│   └── figures/                   # exported plots
├── src/
│   ├── data/
│   │   └── make_dataset.py        # load, merge, resample sensor data
│   ├── features/
│   │   └── remove_outliers.py     # outlier detection and removal
│   ├── models/
│   └── visualization/
│       └── visualize.py           # exploratory plots
├── environment.yml
├── requirements.txt
└── README.md
```

The layout follows the Cookiecutter Data Science template.

## Getting Started

**1. Clone the repository**

```bash
git clone https://github.com/serafimovskateodora/FitnessTracker-.git
cd FitnessTracker-
```

**2. Install dependencies**

```bash
pip install -r requirements.txt
```

or, with conda:

```bash
conda env create -f environment.yml
```

**3. Add the data**

Place the raw MetaMotion CSV files in `data/raw/MetaMotion/`.

**4. Run the scripts in order**

```bash
python src/data/make_dataset.py              # merge + resample  -> data/interim/01_data_processed.pkl
python src/features/remove_outliers.py       # outlier removal   -> data/interim/02_outliers_removed_chauvenets.pkl
python src/visualization/visualize.py        # plots             -> reports/figures/
```

> The scripts use relative paths (`../../data/...`), so run them from inside their own folder (for example `cd src/data`) or from your IDE.

## Results and Figures

[Add 1-2 screenshots from `reports/figures/`, for example the combined accelerometer and gyroscope plot for one exercise.]

```markdown
![Squat (A)](reports/figures/Squat%20(A).png)
```

## Key Learnings

- Working with **multi-rate time-series data** and aligning it on a common time grid.
- Parsing metadata from file names to label data automatically.
- The difference between **distribution-based** and **distance-based** outlier detection, and why the right choice depends on how the data is distributed.
- Why preprocessing must happen **per group** (per exercise) when groups behave differently.

## Next Steps

- [ ] Low-pass filtering to reduce noise
- [ ] Feature engineering (temporal, frequency-domain features)
- [ ] Train a classifier to recognize exercises
- [ ] Repetition counting

## Acknowledgements

- The outlier detection functions (Chauvenet's criterion and the outlier plotting helper) are adapted from the [ML4QS](https://github.com/mhoogen/ML4QS) repository.