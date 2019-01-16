
# Who are Starbucks's Coupon-Savvy Customers?


## 1. Project Overview
More and more companies are leveraging customer data analysis to improve user experience. In this project, we will do data analysis on a dataset from Startbucks and try to answer the question of who likes what offers from Starbucks.
### 1.1 Problem Statement
Starbucks sends out offers to its customers once in a while. The offer can be a discount, a BOGO or just informational. Also each offer has a minimal expense and a valid period.

Each customer gets different offers on different times. Since every customer has different interests and responds differently to different offers, we need to find out which demographic groups respond best to which offer type, so that customers can benefit most from the offers they receive.

The result of the data analysis process in this project will provide a set of heuristics that can serve as a guidance when sending out offers to customers.

### 1.2 Dataset
The dataset is provided by Starbucks in three files:

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

## 2. Data Pre-Processing
### 2.1 Customer data in Profile.json
 We noticed some abnormalities in customer Profile data. There are 2175 entries with age = 118. According to Wikipedia: https://en.wikipedia.org/wiki/List_of_the_verified_oldest_people, the current oldest living person is 116 years old. Also, all these entries have no information for gender and income. Considering those data entries will not provide any value to our analysis, we decide to delete those entries from Profile data.

 The membership data for the customers are recorded as the date the customer became a member. In later analysis, we'll need the data for how long the customer has been a member, so we convert the value of **became_member_on** to **member_months** which is the number of months the customer has been a member till 2018-12-28.

 Also, the 32-digit Hexadecimal numbers are not very easy to use as customer ids. We will use row index number of Profile data as customer id for convenient.

### 2.2 Offer data in Portfolio.json
Same as customer id, the offer ids are also 32-digit Hexadecimal numbers. Again we replaced them with simple integer numbers for easy identification in later analysis.

Since the time data in **transcript.json** are recorded in hours, we will convert the offer valid period, which is in days, to hours.

### 2.3 Customer Activity data in Transcript.json
There are two types of activities in **transcript.json**. One type is offer-related such **offer_received**, **offer_viewed** and **offer_completed**. The other type is the actual purchase. We need to separate these two types of data, because they need to be processed differently.


For offer-related activities, we need to extract offer_id from **value** column. Then we need to map the Hexidecimal offer_id to the re-coded simple integer **o_id**. Also, the Hexidecimal customer_id needs to be mapped to the re-coded simple integer **c_id**.

We notice that after customer_id mapping, some offer activities have NaN in **c_id**. This is because some customers in offer transcript are not in customer profile. We will delete those rows since there's no customer data associated with these records. There are about 11% rows in offer activity data that are deleted because of missing customer data.


## 3. Data Exploration
### 3.1 Customer Distribution
The following charts show customer distributions by **gender**, **age**, **income** and **member_months**.

### 3.2 Offer Distribution
There are 10 different offers in Portfolio data file. The following chart shows for each offer how many are received, viewed and completed.

## 4. Data Analysis
### 4.1 Divide Customers into groups
According to the customer distribution chart, we can divide age into 3 groups: **<=40**, **41-70** and **>71**. Customer income  can be divided into 3 groups:**<=50000**, **50001 to 80000** and **>=80001**.  Membership in months can also be divided into 3 groups: **<=1_year**, **1_to_3_years** and **>3_years**. Gender data are naturally in 3 categories: F, M and O(unidentified). The customer profile data after data transformation look like this:

### 4.2 Identify viewed and completed offers in received offers
For each received offer, we need to determine whether it has been viewed and/or completed. To determine if an received offer has been viewed, we take the (c_id, o_id) pair as key and search in the offer_viewed activities. If we can find matching records in offer_viewed activities and the time stamp in within the valid period of the received offer, then we'll set **viewed** column as **True** for that received offer The same process applies to determining completed offers.

After **viewed** and **complete** columns are set for all received offers, we want to validate whether the numbers of identified viewed offers and completed offers  
match the numbers of offer_viewed activities and offer_completed activities.

As we can see above, the offers that can be identified as viewed or complete are less than the real number of completed or viewed offers. Since the difference is small, about 4%, we'll consider this method of identifying viewed and completed offers in received offers as valid.

The combined data with customer profile looks like this:

### 4.3 Heatmaps
With the combined data, we can create heatmaps of viewed percentage of each offer type by customer groups.

As we can see, offer #2, offer #6, offer #7 and offer #9 have higher percentage of being viewed for almost all customer groups by any individual demographic characteristic(i.e. age alone, income alone or gender alone).

The following heatmaps show the percentage of completed offers for each offer type by Customer Groups. Only offer #6 and offer #7 have obvious higher percentage of being completed for certain customer groups, namely age > 40, femail, income > 80000 and membership > 3 years.

### 4.4 Best responded offer for each customer group
But the heapmaps cannot tell us given a customer group described by all 4 characteristics in the profile (e.g. age > 40 and male with income > 80000 and membership < 1 year), which offer has the highest rate of being viewed or completed.

We then generate a table with percentage of viewed offers and percentage of completed offers for each offer type for each customer group that can be described by all 4 characteristics in the customer profile. The table looks like this:

With this table, for any given customer group, we can query for the percentage of viewed offers and percentage of completed offers for each offer type. Then we can choose which offer type is the best to sent to this customer group. It can be the offer type with the highest completed percentage or highest viewed percentage.

For example, if we want to decide which offer to send to a female customer with age between 41 and 70, income between 50000 to 80000 and membership between 1 to 3 years, we can do the following query on the table.

The result shows that offer #7 is the best for this customer, because offer #7  has the highest completed percentage and second highest viewed percentage for this particular customer group.  
### 4.5 Customer groups that are least influenced by offers
We would also like to find out which customer groups are less influence by the offers. These are the customers who will likely to make purchases even without any offers.

From the raw Transcript.json data, we select the activities that are actual purchases(event == transaction). For each purchase record, we identify whether it is influenced by an offer or not. This is done by matching the time stamp of the purchase activity with the offer valid period. If the time of the purchase activity is within an offer's valid period and this offer has been viewed by the same customer, then we assume this purchase is influenced by the offer.

## 5. Results

## 6. Conclusions  
improve the process of identifying viewedd and complete.
