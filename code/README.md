# Buy or Wait?

An AI-assisted financial decision system built for the HackerRank Orchestrate September 2026 hackathon.

## Overview

**Buy or Wait?** is a financial decision agent that evaluates whether a user can safely afford a requested expense.

A decision cannot be based on the user's current balance alone. The system considers the user's financial profile, historical and scheduled transactions, recurring income and expenses, pending payments, minimum-balance requirements, available payment options, and relevant financial information contained in messages and images.

For every request, the system determines an appropriate financial outcome, including whether the user should:

- Pay in full
- Make a partial payment
- Use installments
- Wait until the purchase becomes affordable
- Avoid the purchase

The resulting recommendation is constrained by the user's financial commitments, preferences, and minimum balance requirements.

---

## Setup

### Requirements

- Python 3.10 or newer
- Windows, macOS, or Linux
- No external API key is required for the final deterministic solver

### 1. Clone or extract the project

If using Git:

```bash
git clone <your-repository-url>
cd <repository-directory>
```

Alternatively, extract the submitted ZIP and open a terminal in the project root.

### 2. Create a virtual environment

Creating a virtual environment is recommended.

**Windows PowerShell**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

The final solver uses Python's standard library and does not require third-party packages for normal execution.

No package installation is required.

### 4. Verify the project structure

Run commands from the repository root. The `dataset/` directory must be present because the solver reads the challenge inputs relative to the repository root.

Expected structure:

```text
repository/
├── code/
│   ├── main.py
│   └── evaluation/
│       └── usage_report.md
├── dataset/
├── output.csv
├── README.md
├── AGENTS.md
├── CLAUDE.md
└── problem_statement.md
```

The input datasets must remain in their supplied locations under `dataset/`.

---

## Solution Approach

The solution is implemented in Python in `code/main.py`.

The system follows a deterministic financial-planning pipeline:

1. Load the supplied datasets from `dataset/`.
2. Reconstruct the user's current financial state.
3. Resolve historical, pending, scheduled, and confirmed financial events.
4. Identify recurring income and expense patterns.
5. Incorporate relevant information from messages and supplied image evidence.
6. Forecast the user's financial position over the required horizon.
7. Calculate the amount that can safely be paid while protecting essential expenses and the user's minimum balance.
8. Generate feasible payment strategies.
9. Evaluate partial-payment and installment options against the supplied constraints.
10. Consider permitted spending changes where applicable.
11. Select the safest applicable recommendation according to the user's request and preferences.
12. Validate the generated result before writing it to `output.csv`.

The implementation keeps financial reconstruction, forecasting, plan generation, and output validation separate so that generated recommendations must satisfy the financial safety constraints.

---

## Input Data

The solution reads the supplied files from the `dataset/` directory.

Important inputs include:

| File | Purpose |
|---|---|
| `requests.csv` | The 250 requests that must be evaluated |
| `sample_requests.csv` | 25 labelled examples used for validation |
| `financial_profiles.csv` | Balances, minimum balance, priorities, and preferences |
| `financial_events.csv` | Historical, pending, scheduled, and confirmed financial events |
| `request_payment_options.csv` | Payment options available for each request |
| `exchange_rates.csv` | Fixed exchange rates required for currency conversion |
| `messages.csv` | Relevant financial messages |
| `images.csv` | Metadata linking financial evidence to events or requests |
| `media/images/` | Supplied image evidence |

The solution does not require live banking access, live exchange rates, or external market data.

---

## Running the Solution

Run the solution from the repository root:

```bash
python code/main.py
```

This reads the supplied datasets and generates:

```text
output.csv
```

in the repository root.

### Validate the labelled samples

The 25 labelled requests can be checked using:

```bash
python code/main.py --validate-samples
```

This compares the solver's results against the expected results in:

```text
dataset/sample_requests.csv
```

---

## Output

The final `output.csv` contains one prediction for every request in `dataset/requests.csv`.

The required columns are:

```text
request_id
amount_safe_to_pay
affordability_status
recommended_payment_method
payment_plan
earliest_date_for_full_payment
spending_changes_needed
decision_explanation
```

### Output fields

| Field | Description |
|---|---|
| `request_id` | Request being evaluated |
| `amount_safe_to_pay` | Largest amount safe to pay on the request date before optional spending changes |
| `affordability_status` | `affordable_now`, `affordable_with_plan`, `affordable_later`, or `not_affordable` |
| `recommended_payment_method` | `full_payment`, `partial_payment`, `installments`, `wait`, or `not_recommended` |
| `payment_plan` | Chronological payment entries or `none` |
| `earliest_date_for_full_payment` | Earliest date when the full requested amount can safely be paid |
| `spending_changes_needed` | Permitted spending changes or `none` |
| `decision_explanation` | Concise explanation of the financial decision |

The fundamental safety constraint is:

```text
0 <= amount_safe_to_pay <= requested_amount
```

Installment plans must correspond to a supplied payment option, and spending changes can only target permitted flexible recurring expenses.

---

## Validation Results

The final solver was evaluated against the 25 labelled samples.

### Field-level results

| Field | Exact matches | Accuracy |
|---|---:|---:|
| `recommended_payment_method` | 23 / 25 | 92% |
| `affordability_status` | 22 / 25 | 88% |
| `payment_plan` | 22 / 25 | 88% |
| `spending_changes_needed` | 20 / 25 | 80% |
| `earliest_date_for_full_payment` | 19 / 25 | 76% |
| `amount_safe_to_pay` | 7 / 25 | 28% |

### Combined result

The combined status/method criterion requires both `affordability_status` and `recommended_payment_method` to match.

**Combined status/method matches: 19 / 25 (76%)**

### Full 250-request output audit

The final generated output contains **250 request rows**.

The output was checked for structural and mechanical consistency:

| Check | Result |
|---|---:|
| Output data rows | 250 |
| Required request IDs represented exactly once | Pass |
| Missing IDs | 0 |
| Duplicate IDs | 0 |
| Extra IDs not in `requests.csv` | 0 |
| Required columns present and exact order | Pass |
| Invalid affordability-status enum values | 0 |
| Invalid payment-method enum values | 0 |
| Safe amounts nonnumeric, negative, or above requested amount | 0 |
| Empty decision explanations | 0 |
| Malformed spending-change expressions | 0 |
| More than three spending changes | 0 |
| Malformed, nonpositive, or nonchronological payment plans | 0 |
| Partial-payment mechanical consistency errors | 0 |
| Full-payment/wait plan count errors | 0 |
| Installment plans not matching a supplied installment option | 0 |
| Plans incorrectly present for `not_recommended` rows | 0 |

The 250-request output passed the structural and consistency audit. No accuracy percentage is reported for those 250 rows because the evaluation set does not have publicly available ground-truth labels.

---

## Project Structure

```text
.
├── code/
│   ├── main.py
│   └── evaluation/
│       └── usage_report.md
│
├── dataset/
│   ├── requests.csv
│   ├── sample_requests.csv
│   ├── financial_profiles.csv
│   ├── financial_events.csv
│   ├── request_payment_options.csv
│   ├── exchange_rates.csv
│   ├── messages.csv
│   ├── images.csv
│   └── media/
│       └── images/
│
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── log.txt
├── output.csv
└── problem_statement.md
```

---

## Model / API Usage

The final submitted solver is a deterministic Python implementation and does not require an external API or API key to execute.

Model/API usage information for the final full-dataset run is documented separately in:

```text
code/evaluation/usage_report.md
```

No API keys, credentials, or sensitive configuration are included in the submission.

---

## Reproducibility

The solution is designed to run from the repository root using the supplied challenge datasets.

It reads its input data from:

```text
dataset/
```

and writes the final predictions to:

```text
output.csv
```

The solution does not depend on live financial accounts, live exchange rates, live market data, or external banking services.

---

## Submission Artifacts

The HackerRank submission consists of:

### `code.zip`

Contains the complete runnable solution, including:

- Source code
- `README.md`
- Setup and run instructions
- `evaluation/usage_report.md`
- Required project files

### `output.csv`

Contains predictions for all 250 requests in:

```text
dataset/requests.csv
```

The file contains one prediction row per request plus the header.

### `chat_transcript`

The required development transcript generated from the project's `log.txt`.

Before submission, confirm that the runnable code, README, evaluation folder, and required output files are present in the submitted package.

---

## Notes

This project was developed specifically for the HackerRank Orchestrate **Buy or Wait?** challenge and is intended to operate against the datasets and specification supplied with the challenge.
