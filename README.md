# Starbucks
### 1 Overview
Starbucks sends out offers to its customers once in a while. The offer can be a discount, a BOGO or just informational. Also each offer has a minimal expense and a valid period.

Each customer gets different offers on different times. Since every customer has different interests and responds differently to different offers, we need to find out which demographic groups respond best to which offer type, so that customers can benefit most from the offers they receive.

The result of the data analysis process in this project will provide a set of heuristics that can serve as a guidance when sending out offers to customers.

### 2 Dataset
The dataset is under /data folder in three files:

1. **portfolio.json** - containing following offer information:
  - id (string) - offer id
  - offer_type (string) - type of offer i.e. BOGO, discount, informational
  - difficulty (int) - minimum required spend to complete an offer
  - reward (int) - reward given for completing an offer
  - duration (int) - valid period in days
  - channels (list of strings) - how the offer is sent i.e. web, email, mobile, social

2. **profile.json** - demographic data for each customer
  - age (int) - age of the customer
  - became_member_on (int) - date when customer created an app account
  - gender (str) - gender of the customer (some entries contain 'O' for other rather than M or F)
  - id (str) - customer id
  - income (float) - customer's income
3. **transcript.json** - records for transactions
  - event (str) - record description (ie transaction, offer received, offer viewed, etc.)
  - person (str) - customer id
  - time (int) - time in hours. The data begins at time t=0
  - value - (dict of strings) - either an offer id or transaction amount depending on the record

### 4 Code Files
  - Starbucks_Capstone_notebook.ipynb - the main file to do analysis and generate results
  - /data/JSON_to_CSV.ipynb - converts transcript.json to transcript.csv, so that I don't need to run 'conda update pandas' in the terminal every time I start the workspace.
### 5 Instructions
To run the notebook Starbucks_Capstone_notebook.ipynb, the following Python packages are needed:
1. numpy
2. pandas
3. matplotlib
4. seaborn
5. math
6. json
7. datatime
8. time

### 6 Blog
The blog presenting this analysis can be found at (https://cr8zysheila.github.io/Starbucks/).
