# Data provenance and dictionary

Both existing datasets are preserved unchanged. These checks describe the local copies; no upstream download was used to establish identity.

| File | Shape | Target counts | Experiment |
| --- | --- | --- | --- |
| `default of credit card clients.xls` | 30,000 rows, 25 columns | 23,364 non-default; 6,636 default | A: notebooks 01-03 |
| `credit.csv` | 24,000 rows, 24 columns | 18,694 non-default; 5,306 default | B: notebook 04 |

Read the Excel with `header=1`: its first row contains extra labels. It includes ID, 23 predictors and `default payment next month`. The CSV has no ID, calls the latest repayment variable PAY_1 and the target `default`. Notebook 04 renames these to PAY_0 and DEFAULT; all notebooks rename the target to DEFAULT.

The Excel is attributed to [UCI Default of Credit Card Clients](https://archive.ics.uci.edu/dataset/350/default+of+credit+card+clients). Notebook 04 attributes the CSV to [PyCaret datasets](https://github.com/pycaret/datasets/blob/main/data/common/credit.csv), with a legacy fallback URL retained in code. The exact original CSV extraction procedure is undocumented. After normalizing names and numeric types, all 24,000 CSV rows match rows in the included Excel. This does not recover stable client IDs or establish how the subset was selected.

## Variables

| Variable | Meaning / treatment |
| --- | --- |
| ID | Excel client identifier; excluded from models |
| LIMIT_BAL | Credit limit, New Taiwan dollars (NT$) |
| SEX | 1 male, 2 female; categorical in A, excluded in B |
| EDUCATION | 1 graduate school, 2 university, 3 high school, 4 other; additional codes 0, 5, 6 retained separately |
| MARRIAGE | 1 married, 2 single, 3 other; additional code 0 retained; categorical in A, excluded in B |
| AGE | Age in years |
| PAY_0, PAY_2, ..., PAY_6 | Repayment status, September through April 2005, latest first; CSV uses PAY_1 for September |
| BILL_AMT1, ..., BILL_AMT6 | Monthly statement amounts, NT$, latest first |
| PAY_AMT1, ..., PAY_AMT6 | Monthly payment amounts, NT$, latest first |
| DEFAULT | Next-month default: 1 default, 0 non-default |

Repayment documentation describes -1 as paid duly and positive values as months of delay. Values -2 and 0 also occur; their precise meaning is not established by this project. Informal interpretations are not verified definitions. Existing models retain these codes as numeric inputs. The Excel intentionally jumps from PAY_0 to PAY_2; there is no PAY_1 column.

No values are missing in either file. Negative bill amounts are retained. There are 35 duplicate Excel rows after excluding ID, and 24 duplicate CSV rows. This does not establish errors or independence; no deduplication was performed. Possible duplicate overlap across train/test needs further assessment.

## Preservation and reuse

Hashes and verification are in `../RESULTS.md`. No cleaning, resampling or data conversion was performed during repository maintenance. Use the included CSV rather than the download fallback for the recorded experiment.

The repository does not establish an independent data redistribution license. Consult source terms and attribution requirements before reuse; no new data license is asserted here.
