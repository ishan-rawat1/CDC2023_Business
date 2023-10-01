# Analysis Steps
## 1. Data Importation
The code begins by importing necessary libraries and then uploading a CSV file containing the dataset. This is done using Google Colab's **'files.upload()'** method.

```python
import numpy as np
import matplotlib.pyplot as plt
from google.colab import files
import io
import pandas as pd
import re


data = files.upload()
```
## 2. Data Preprocessing
* The code contains a custom function **'custom_date_converter'** that handles date conversion. We changed the date format from **'01st Jan 88'** to **'01/01/1988'**. This function isn't utilized in the active code, but it's prepared to reformat date strings to a more standard format.
* Data is read into a pandas dataframe, **'df'.**
```python
# import pandas as pd         /// Pycharm (pip install pandas openpyxl)
# import re



# def custom_date_converter(date_str):
#     if not isinstance(date_str, str):
#         return date_str
#     """
#     Converting date string from format '01st Jan 88' to 'mm/dd/year'
#     """
#     # Replace st, nd, rd, th with ''
#     date_str = re.sub(r'(st|nd|rd|th)', '', date_str)
#     # Converting to datetime object
#     date_obj = pd.to_datetime(date_str, format='%d %b %y')
#     # Converting datetime object to desired format
#     return date_obj.strftime('%m/%d/%Y')

# file_path = 'Business_Dataset.xlsx'
# df = pd.read_excel(file_path, engine='openpyxl')

# # Checking if columns 'Release Date' and 'Last Update' exist
# if 'Release Date' in df.columns:
#     df['Release Date'] = df['Release Date'].apply(custom_date_converter)

# if 'Last Update' in df.columns:
#     df['Last Update'] = df['Last Update'].apply(custom_date_converter)

# # Saving modified dataframe back to Excel
# df.to_excel(file_path, index=False, engine='openpyxl')
```
## 3. Listing Games by Publisher
* We grouped the data by 'Publisher' and listed the games associated with each publisher.
* Displayed the publishers and their games.
```python
grouped = df.groupby('Publisher')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Publisher: {row['Publisher']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")
```
## 4. Counting Games by Publisher
* Created a dictionary **('publisher_counts_dict')** to store the number of games each publisher has.
* Sorted the publishers by game count in descending order and displayed the count.
```python
publisher_counts_dict = {}
for publisher in df['Publisher']:
    if publisher in publisher_counts_dict:
        publisher_counts_dict[publisher] += 1
    else:
        publisher_counts_dict[publisher] = 1
sorted_publisher_counts = sorted(publisher_counts_dict.items(), key=lambda item: item[1], reverse=True)
for publisher, count in sorted_publisher_counts:
    print(f"Publisher: {publisher}, Game Count: {count}")
```
## 5. Computing Average Critic Scores by Publisher
* Grouped the data by 'Publisher' and gathered the 'Critic Score' for each.
* Filtered out invalid or missing scores.
* Computed the average critic score for each publisher.
* Sorted the publishers by the average score in descending order and displayed the results.
```python
grouped1 = df.groupby('Publisher')['Critic Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each publisher
avg_scores_dict = {}

for index, row in grouped1.iterrows():
    # Filter out invalid scores
    valid_scores = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores:
        avg = sum(valid_scores) / len(valid_scores)
        avg_scores_dict[row['Publisher']] = (avg, len(valid_scores))

# Sorting publishers by their average scores in descending order
sorted_publishers = sorted(avg_scores_dict.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for publisher, (avg, count) in sorted_publishers:
    print(f"Publisher: {publisher}")
    print("The average critic values for each publisher is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")
```
## 6. Computing Average User Scores by Publisher
* Similarly, grouped the data by 'Publisher' and gather the 'User Score' for each.
* Filtered out invalid or missing scores.
* Computed the average user score for each publisher.
* Sorted the publishers by average score in descending order and display the results.
```python
grouped1 = df.groupby('Publisher')['User Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each publisher
avg_scores_dict = {}

for index, row in grouped1.iterrows():
    # Filter out invalid scores
    valid_scores = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores:
        avg = sum(valid_scores) / len(valid_scores)
        avg_scores_dict[row['Publisher']] = (avg, len(valid_scores))

# Sorting publishers by their average scores in descending order
sorted_publishers = sorted(avg_scores_dict.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for publisher, (avg, count1) in sorted_publishers:
    print(f"Publisher: {publisher}")
    print("The average user values for each publisher is:", round(avg, 2))
    print(f"Number of critic scores: {count1}")
    print("\n")
```
## 7. Calculating Difference between Critic and User Scores:
* Filtered the dataset to only include rows where both 'Critic Score' and 'User Score' are available.
* Calculated the absolute difference between the two scores for each game.
* Sorted by this difference in descending order and displayed the top 10 games with the highest difference.

```python
# Filtering the dataframe for rows where both 'Critic Score' and 'User Score' are not NaN
valid_df = df.dropna(subset=['Critic Score', 'User Score']).copy()

# Calculating the absolute difference between 'Critic Score' and 'User Score'
valid_df.loc[:, 'Difference'] = abs(valid_df['Critic Score'] - valid_df['User Score'])

# Sorting the dataframe by the 'Difference' column in descending order and getting the top 10 rows
top_10_diff = valid_df.sort_values(by='Difference', ascending=False).head(10)

# Displaying the publishers with the maximum difference
print(top_10_diff[['Publisher', 'Difference']])
```








