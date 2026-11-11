# Amygdala subregion functional connectivity, aging, and resilience — data and analysis configuration

This repository contains the data and the analysis configuration used to relate voxel-wise amygdala subregion functional connectivity (FC) to emotional and cognitive resilience in younger and older adults, using behavioral Partial Least Squares (PLS).

> **Citation:** [Authors]. [Title]. [Journal], [Year]. [DOI]
> Please cite this paper if you use these data or this code.

---

## Contents

| File | Size | Description |
|---|---|---|
| `beh_osf.csv` | 16 KB | Demographic, behavioral, and amygdala subregion measures for the 134 participants included in the analysis |
| `ROI1.mat` | ~287 MB | Voxel-wise FC maps for amygdala seed ROI 1 |
| `ROI2.mat` | ~286 MB | Voxel-wise FC maps for amygdala seed ROI 2 |
| `ROI3.mat` | ~286 MB | Voxel-wise FC maps for amygdala seed ROI 3 |
| `myPLS_configure_v1.m` | 11 KB | Configuration script for the behavioral PLS analysis (myPLS toolbox) |

---

## 1. `beh_osf.csv`

One row per included participant (N = 134). There are no missing values.

| Group | Code | n | Age range (years) | Sex (M / F) |
|---|---|---|---|---|
| Younger adults (YA) | 1 | 68 | 19–36 | 36 / 32 |
| Older adults (OA) | 2 | 66 | 65–82 | 32 / 34 |

### Variables

| Column | Description |
|---|---|
| `Subject` | Participant ID |
| `Group` | Age group: `1` = younger adults (YA), `2` = older adults (OA) |
| `Age` | Age in years |
| `Order` | Row index of this participant in the `voxel_fc` matrices in `ROI*.mat` (1–141; see Section 2) |
| `Sex` | `1` = male, `2` = female |
| `Gender` | `M` / `F` (same information as `Sex`) |
| `EV_base`, `EV_neu`, `EV_neg` | [Rating EV, e.g. emotional valence], 1–9 scale, at baseline, after the neutral film, and after the negative film |
| `ER_base`, `ER_neu`, `ER_neg` | [Rating ER, e.g. emotional arousal], 1–9 scale, at baseline, after the neutral film, and after the negative film |
| `rSFA`, `rCMA`, `rBLA` | Right superficial (SFA), centromedial (CMA), and basolateral (BLA) amygdala [measure and unit, e.g. volume in mm³] |
| `lSFA`, `lCMA`, `lBLA` | Left SFA, CMA, and BLA [measure and unit] |
| `Emo_res` | Emotional resilience score [how it was derived] |
| `Cog_res` | Cognitive resilience score [how it was derived] |

`Emo_res` and `Cog_res` are the two behavioral variables entered into the PLS (`behav_names = {'Emotion','Cognition'}`).

---

## 2. `ROI1.mat`, `ROI2.mat`, `ROI3.mat`

Each file holds one variable:

| Variable | Dimensions | Type |
|---|---|---|
| `voxel_fc` | 141 × 570,380 | single |

- **Rows (141)** are participants. The matrices hold all 141 scanned participants; 7 of them were excluded from the analysis and do not appear in `beh_osf.csv`. Use the `Order` column to select and align the 134 included participants:

  ```matlab
  beh = readtable('beh_osf.csv');
  load('ROI1.mat');                       % loads voxel_fc
  X = voxel_fc(beh.Order, :);             % 134 x 570380, same row order as beh
  ```

- **Columns (570,380)** are voxels: FC between the seed ROI and each voxel, [e.g. Fisher z-transformed Pearson correlation]. Values range from about −3.1 to 4.5. About 19,900 columns are zero for every participant (voxels outside the data coverage) and can be dropped before analysis.
- **Seed ROIs:** ROI1 = [BLA / CMA / SFA], ROI2 = [ ], ROI3 = [ ]; [hemisphere: bilateral / left / right].
- **Voxel space:** [template, resolution, and the mask that defines the voxel order]. The mask is needed to map the vectors back into brain space. It is [included / available from …].

---

## 3. `myPLS_configure_v1.m`

This script sets the inputs and options for a behavioral PLS correlation analysis with the [myPLS toolbox](https://github.com/danizoeller/myPLS) (MIP:lab). It is called by the toolbox's main script (`myPLS_main.m`).

### Inputs

| Input | Setting |
|---|---|
| Brain data `X` | `voxel_fc_reg`: voxel-wise FC for one seed, [after regressing out nuisance variables, e.g. sex] |
| Behavior data `Y` | `PLS_beh`: `Emo_res` and `Cog_res` |
| Grouping | `beh.Group` (YA, OA) |

### PLS options

| Option | Value |
|---|---|
| Analysis type | `behavior` (standard behavioral PLS) |
| Permutations / bootstraps | 1000 / 500 |
| Normalization (imaging and behavior) | z-score within groups |
| Grouped PLS, permutations, and bootstrapping | yes (within group) |
| Procrustes rotation for bootstrapping | average rotation of U and V |
| Significance level for LCs | α = 0.05 |
| Bootstrap ratio display threshold | ±3.3 |
| Loading display threshold | ±0.4 |

### Variables to define before running

The script does not load data itself. These variables must exist in the workspace before it runs:

- `voxel_fc_reg`: brain data matrix (participants × voxels)
- `PLS_beh`: behavior matrix (participants × 2)
- `beh`: the table from `beh_osf.csv`
- `output_path`, `prefix`: where results are saved and their file prefix
- `img_type`: `'volume'`, `'corrMat'`, or `'barPlot'`
- `mask_file`, `struct_file`: brain mask and structural template (needed for `'volume'` plots)

### Example workflow

```matlab
addpath(genpath('path/to/myPLS'));

beh = readtable('beh_osf.csv');
load('ROI1.mat');                              % voxel_fc
X = voxel_fc(beh.Order, :);
X(:, all(X == 0, 1)) = [];                     % optional: drop empty voxels (adjust mask accordingly)

voxel_fc_reg = X;                              % [apply nuisance regression here if used]
PLS_beh      = [beh.Emo_res, beh.Cog_res];

output_path = 'results/ROI1';
prefix      = 'ROI1';
img_type    = 'volume';
mask_file   = '[mask.nii]';
struct_file = '[template.nii]';

myPLS_main;                                    % runs myPLS_configure_v1 internally
```

---

## Requirements

- MATLAB [version]
- [myPLS toolbox](https://github.com/danizoeller/myPLS)
- About 1.2 GB of memory per ROI file once loaded (single precision)

## Data acquisition and preprocessing

[Dataset, scanner and field strength, film paradigm, preprocessing pipeline, denoising, and seed definition. Or: "See the Methods section of the paper."]

## License

[e.g. CC BY 4.0 for data; MIT for code]

## Contact

Shuer Ye — [email]
