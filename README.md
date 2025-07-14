
# Metapredictor

Metapredictor is an ensemble deep learning-based predictor package
designed to predict effectors from individual kingdoms such as bacteria,
fungi, oomycete, as well as all the kingdoms at the same time. It
integrates outputs from kingdom-specific prediction programs and applies
a deep learning model to classify

This package processes outputs from kingdom-specific prediction
programs, formats them, merges results, performs encoding, and generates
final predictions using a built-in deep learning model.

------------------------------------------------------------------------

## Table of Contents

- [Overview](#overview)
  - [Features](#features)
- [Recommended but Optional: Use a Virtual
  Environment](#recommended-but-optional:-use-a-virtual-environment)
  - [Using Conda](#using-conda)
  - [Using venv (Standard Python)](#Using-venv-(standard-python))
- [Installation](#installation)  
- [Usage](#usage)
  - [Formatting Results from
    Programs](#formatting-results-from-programs)  
  - [Merging Results](#merging-results)  
  - [Encoding and Scaling](#encoding-and-scaling)  
  - [Making Predictions](#making-predictions)  
- [Function Reference](#function-reference)  
- [Examples](#examples)  
- [Note](#note)
- [Contact](#contact)
- [License](#license)

------------------------------------------------------------------------

## Overview

Metapredictor is designed to streamline effector prediction by combining
outputs from different kingdom-specific prediction tools, standardizing
them into a consistent format, encoding predictions for model
compatibility, and producing a final classification.

### Features

- Format raw outputs from different effector prediction tools into a
  uniform structure.
- Merge multiple formatted prediction results by identifier.
- One-hot encode and reorder merged predictions for compatibility with
  the deep learning model.
- Run deep learning model predictions built into the package.

The package handles multiple kingdoms:

| Kingdom | Programs Supported | Required Columns for Encoding |
|----|----|----|
| bacteria | Deepredeff, EffectiveT3, T3SEpp | `Deepredeff`, `EffectiveT3`, `T3SEpp` |
| fungi | EffectorP, Deepredeff, WideEffHunter | `EffectorP`, `Deepredeff`, `WideEffHunter` |
| oomycete | EffectorP, Deepredeff, WideEffHunter, EffectorO | `EffectorP`, `Deepredeff`, `WideEffHunter`, `EffectorO` |
| all | All programs | `EffectorP`, `EffectiveT3`, `EffectorO`, `Deepredeff`, `WideEffHunter`, `T3SEpp` |

------------------------------------------------------------------------

## **Recommended but Optional: Use a Virtual Environment**

To avoid conflicts with existing packages and ensure reproducibility, it
is highly recommended to install Metapredictor in a fresh virtual
environment.

### Using Conda

``` python

conda create -n metapredictor-env python=3.9
conda activate metapredictor-env
pip install metapredictor
```

### Using venv (Standard Python)

``` python

python -m venv metapredictor-env
source metapredictor-env/bin/activate  \# On Windows: metapredictor-env\Scripts\activate
pip install metapredictor
```

------------------------------------------------------------------------

## Installation

Install the package via pip using:

``` bash
pip install metapredictor
```

------------------------------------------------------------------------

## Usage

### Formatting Results from Different Prediction Programs

Convert raw outputs from various effector prediction tools into a
consistent CSV format. Use the formatting functions under the
`preprocessing` module.

**Example for EffectorP (fungi):**

``` python

from metapredictor.preprocessing import format_effectorp_result

formatted_df = format_effectorp_result(
input_file="raw_data/effectorp_output.txt",
kingdom="fungi",
output_file="formatted/effectorp_fungi.csv"
)
```

**Similarly, use:**

- `format_effectiveT3_result()` for EffectiveT3 (bacteria)
- `format_deepredeff_result()` for Deepredeff (oomycete, bacteria, or
  fungi)
- `format_WideEffHunter_result()` for WideEffHunter (fungi or oomycete)
- `format_T3SEpp_result()` for T3SEpp (bacteria)
- `format_effectoro_result()` for EffectorO (oomycete)
- `format_effectorp_result()` for EffectorP (fungi or oomycete)

------------------------------------------------------------------------

### Merging Results

Combine multiple formatted prediction result files (from different
programs) by their Identifier column into a single dataframe, where each
program’s predictions become separate columns.

Use the function merge_predictions_by_identifier() to combine formatted
DataFrames. This merges predictions by Identifier and adds columns named
after each program’s prediction.

``` python
from metapredictor.preprocessing import merge_predictions_by_identifier
import pandas as pd

# Load formatted CSV files into DataFrames
dfs = [
    pd.read_csv("formatted/effectorp_fungi.csv"),
    pd.read_csv("formatted/effectivet3_bacteria.csv"),
    pd.read_csv("formatted/deepredeff_oomycete.csv"),
    pd.read_csv("formatted/wideeffhunter_fungi.csv"),
    pd.read_csv("formatted/t3sepp_bacteria.csv"),
    pd.read_csv("formatted/effectoro_oomycete.csv")
]

merged_df = merge_predictions_by_identifier(dfs, output_file="merged/merged_predictions.csv")

print(merged_df.head())
```

**This function:**

- Checks for required columns (`Identifier`, `Prediction`, `Program`) in
  each DataFrame.
- Renames the `Prediction` column to the program name.
- Merges all data on `Identifier` using an outer join.
- Optionally saves the merged DataFrame to a CSV.

------------------------------------------------------------------------

### Encoding and Scaling

Prepare the merged data for the deep learning model using
`encode_scale_predictions()`.

**This function:**

- Validates the kingdom and required prediction columns.

- One-hot encodes the prediction columns.

- Adds missing columns needed by the model with zeros.

- Orders columns as expected by the model.

- Saves and returns the encoded DataFrame.

- Use the function:

``` python
from metapredictor.preprocessing import encode_scale_predictions

encoded_df = encode_scale_predictions(
    input_file="merged/merged_predictions.csv",
    output_file="encoded/encoded_predictions.csv",
    kingdom="all"
)
```

### Making Predictions

Make final effector predictions using the built-in deep learning model
embedded within the package.

Run the deep learning model using the encoded data:

``` python
from metapredictor.metapredictor import model_prediction

model_prediction(
    input_file="encoded/encoded_predictions.csv",
    output_file="results/final_predictions.csv"
)
```

input_file: Path to encoded and scaled CSV formatted by
encode_scale_predictions.

output_file: Path to save final predictions. Output includes Identifier
and predicted label (Effector or Non-Effector).

------------------------------------------------------------------------

## Function Reference

| Function | Purpose | When to Use |
|----|----|----|
| `format_effectorp_result` | Format EffectorP raw output | After running EffectorP tool |
| `format_effectiveT3_result` | Format EffectiveT3 raw output | After running EffectiveT3 tool |
| `format_deepredeff_result` | Format Deepredeff raw output | After running Deepredeff tool |
| `format_WideEffHunter_result` | Format WideEffHunter predictions from fasta | After WideEffHunter predictions |
| `format_T3SEpp_result` | Format T3SEpp raw output | After running T3SEpp tool |
| `merge_predictions_by_identifier` | Merge formatted DataFrames by Identifier | After formatting all tools’ outputs |
| `encode_scale_predictions` | One-hot encode and reorder merged predictions | Before running model prediction |
| `model_prediction` | Run the ensemble deep learning model | After encoding predictions |

------------------------------------------------------------------------

## Examples

``` python
# 1. Import functions from the metapredictor package
from metapredictor.preprocessing import (
    format_effectiveT3_result,
    format_deepredeff_result,
    format_T3SEpp_result,
    merge_predictions_by_identifier,
)
from metapredictor.encode import encode_scale_predictions
from metapredictor.prediction import model_prediction

# 2. Format the output from each prediction program (bacteria)
df_effectiveT3 = format_effectiveT3_result("effectiveT3_results.csv", kingdom="bacteria")
df_deepredeff = format_deepredeff_result("deepredeff_results.csv", output_file=None, kingdom="bacteria")
df_t3sepp = format_T3SEpp_result("t3sepp_results.csv", kingdom="bacteria")

# 3. Merge formatted predictions by Identifier
merged_df = merge_predictions_by_identifier([
    df_effectiveT3,
    df_deepredeff,
    df_t3sepp
], output_file="merged_predictions_bacteria.csv")

# 4. Encode the merged data
encoded_df = encode_scale_predictions("merged_predictions_bacteria.csv", "encoded_predictions_bacteria.csv", kingdom="bacteria")

# 5. Run final prediction using the deep learning model
model_prediction("encoded_predictions_bacteria.csv", "final_predictions_bacteria.csv")
```

------------------------------------------------------------------------

## Notes

- Input files must be formatted using the provided formatting functions
  before merging or encoding.
- Kingdom names are case-insensitive but should be valid options:
  `"bacteria"`, `"fungi"`, `"oomycete"`, `"all"`.
- The deep learning model is built into the package; users don’t need to
  provide or load it separately.
- Extra columns required by the model but missing from the data are
  added during encoding to maintain compatibility. Only ensure you have
  used the right programs, the function will adjust the data to suit the
  model.

------------------------------------------------------------------------

## Contact

For issues, please open an issue on the [GitHub
repository](https://github.com/LoveBio/Effector_prediction/).

------------------------------------------------------------------------

## License

MIT License © 2025 Love Odunlami
