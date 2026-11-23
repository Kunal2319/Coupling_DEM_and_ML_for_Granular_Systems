# Coupling DEM and Machine Learning for Granular Systems

Bachelor's thesis project (Chemical Engineering, IIT Roorkee, 2024–25). I used Discrete Element Method (DEM) simulations to study how the shape of elongated particles changes their alignment in a vibrated bed, and trained a Random Forest to predict a particle's orientation from its position and aspect ratio.

| | |
|---|---|
| **Question** | How does particle aspect ratio shape the orientation distribution in a vibrated granular bed, and how much of a particle's orientation can be predicted from where it is and how elongated it is? |
| **Particles** | Elliptical cylinders, aspect ratios from 0.5 to 10, simulated at an initial, a mid and a final stage |
| **Data** | About 3.1 million particle rows from DEM runs (position, aspect ratio, orientation angle, 0–180°). Not included here because of size |
| **Methods** | DEM simulation, orientation from rotation-matrix data, probability distribution curves, an anisotropy index, Random Forest regression |
| **Stack** | Python, pandas, scikit-learn, matplotlib |
| **Supervisor** | Prof. Anshu Anand, Department of Chemical Engineering, IIT Roorkee |
| **Report** | [`docs/Bachelor's_Thesis_report_Kunal_Mishra.pdf`](docs/Bachelor's_Thesis_report_Kunal_Mishra.pdf) |

## What I found

**Simulation study (the main result of the thesis).**
- Particles near AR ≈ 1 show a single dominant orientation peak (roughly 70–85°). As the aspect ratio rises above about 3, a second peak appears, and elongated particles (AR 4–10) have a more spread-out, less uniform orientation distribution.
- An anisotropy index (the difference between the largest and smallest eigenvalues of the orientation distribution) rises with aspect ratio and changes with height in the bed, with boundary effects visible near the bottom and the top.

| | |
|---|---|
| ![Orientation probability distribution](figures/orientation_probability_distribution.png) | ![Mean orientation and variance vs aspect ratio](figures/orientation_stats_vs_aspect_ratio.png) |
| Orientation probability distribution (see report §4.2 for the stage and aspect ratio) | Mean orientation and orientation variance against aspect ratio |

**Machine learning part.** A Random Forest (50 trees) predicts orientation from `(x, y, z, aspect ratio)`.

| Metric on a random 80/20 split of 3,082,565 rows | Value |
|---|---|
| R² | **0.69** |
| Share of predictions within ±35% of the true angle | **0.83** |

A correction, since it matters when reading the report: the report's table lists "R² Score 0.833". That number is the **within-±35%** figure above, which my notebook computed with a function named `r2_with_tolerance`. The R² of the same model is 0.69. The notebook in this repository reports each metric under its real name.

## Limitations (what I would do differently now)

- **Random split.** Particles from the same simulation share a stage, a shape and a spatial layout, so a random split lets the model see near-copies of its test particles. The 0.69 is probably optimistic. The notebook adds a grouped split that holds out whole simulation files, and a "predict the mean" baseline for comparison.
- **The tolerance metric is weak.** Orientation is an angle on 0–180°, where 0° and 180° are the same alignment, but a relative error `|y − ŷ|/y` treats them as far apart, and it depends on the size of the true angle. The notebook also reports a wrap-around (circular) error.
- **Tree depth.** The report lists a maximum depth of 30. The notebook uses scikit-learn's default depth, and this repository follows the notebook.
- **Importances.** Impurity-based feature importances can favour continuous variables. A permutation importance on held-out files would be a better check.

## Ideas for continuing

- Predict `(sin 2θ, cos 2θ)` instead of the raw angle so the wrap-around is respected, and predict the whole orientation distribution rather than single angles.
- Evaluate on held-out aspect ratios (can the model generalise to a shape it has not seen?) and held-out stages.
- Add contact and neighbour features (local density, number of contacts) instead of position alone.

## Repository layout

```
.
├── README.md
├── requirements.txt
├── notebooks/
│   └── BTP_Orientation_Prediction.ipynb   # loading, cleaning, original split, grouped split, baseline, importances
├── figures/                                # selected figures from the report
├── docs/
│   └── BTP_Report_...pdf                   # full thesis report
└── data/                                   # put the DEM CSV files here (not tracked)
```

## Running it

```bash
pip install -r requirements.txt
# put the particle CSV files in data/  (columns: Points:0, Points:1, Points:2, aspect_ratio, Orientation)
jupyter notebook notebooks/BTP_Orientation_Prediction.ipynb
```

For a quick test on a fraction of the rows, set `BTP_SAMPLE_FRAC=0.1`. To read the data from another folder, set `BTP_DATA_DIR`.

## Contact

**Kunal Mishra**, IIT Roorkee (Chemical Engineering) · [LinkedIn](https://www.linkedin.com/in/kunal-mishra-0b117a228/) · Portfolio: [add link]
