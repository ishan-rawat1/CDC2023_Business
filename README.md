# Analysis Steps
[Original Code (Business Track Data Analysis)](https://colab.research.google.com/drive/1SzMsMZXpk-7cf_PaAplUZvwYN2PaZUeC?usp=sharing)

[Original code (Decade Business Track Data Analysis)](https://colab.research.google.com/drive/12FeP_gnp251kdYqXN9WirafZF51_7Qq-?usp=sharing)

[Original code (5 decades maximum total sales analysis)](https://colab.research.google.com/drive/1GZPCw5SGscJKxjpbOIeCGD63gJX-PVW7?usp=sharing)

[Original code (Console Track Data Analysis)](https://colab.research.google.com/drive/1woxNkUvqCFqtURBsapKmkiiEKIjeVq81?usp=sharing)

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
* For this step we had to use information from open resources and Chat-GPT
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
sorted_publishers = sorted(avg_scores_dict.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)    ## This line of code was made through the information from open sources and Chat-GPT

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



# Decade Business Track Data Analysis
## 1. Loading Datasets
The datasets for each decade (1970s, 1980s, 1990s, 2000s, 2010s, 2020s) are loaded into separate dataframes using the **'pd.read_csv'** method.
```python
df_1970 = pd.read_csv(io.StringIO(data['Business_Dataset(csv)(1970).csv'].decode('utf-8')))
df_1980 = pd.read_csv(io.StringIO(data['Business_Dataset(csv)(1980).csv'].decode('utf-8')))
df_1990 = pd.read_csv(io.StringIO(data['Business_Dataset(csv)(1990).csv'].decode('utf-8')))
df_2000 = pd.read_csv(io.StringIO(data['Business_Dataset(csv)(2000).csv'].decode('utf-8')))
df_2010 = pd.read_csv(io.StringIO(data['Business_Dataset(csv)(2010).csv'].decode('utf-8')))
df_2020 = pd.read_csv(io.StringIO(data['Business_Dataset(csv)(2020).csv'].decode('utf-8')))
```
## 2. Displaying Games by Developer
For each decade, the code groups the dataset by the 'Developer' column and lists all games developed by each developer. This provides a historical view of game production and allows us to see which developers were active during each decade and what games they produced.

```python
grouped = df_1970.groupby('Developer')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Developer: {row['Developer']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")

grouped = df_1980.groupby('Developer')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Developer: {row['Developer']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")

grouped = df_1990.groupby('Developer')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Developer: {row['Developer']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")

grouped = df_2000.groupby('Developer')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Developer: {row['Developer']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")

grouped = df_2010.groupby('Developer')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Developer: {row['Developer']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")

grouped = df_2010.groupby('Developer')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Developer: {row['Developer']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")

grouped = df_2020.groupby('Developer')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Developer: {row['Developer']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")
```
## 3. Counting Games by Developer
For each decade, we counted the number of games developed by each developer. This gives an indication of the most productive developers for each decade.
```python
Developer_counts_dict_1970 = {}
for Developer in df_1970['Developer']:
    if Developer in Developer_counts_dict_1970:
        Developer_counts_dict_1970[Developer] += 1
    else:
        Developer_counts_dict_1970[Developer] = 1
sorted_Developer_counts_1970 = sorted(Developer_counts_dict_1970.items(), key=lambda item: item[1], reverse=True)
for Developer, count in sorted_Developer_counts_1970:
    print(f"Developer_1970: {Developer}, Game Count: {count}")

Developer_counts_dict_1980 = {}
for Developer in df_1980['Developer']:
    if Developer in Developer_counts_dict_1980:
        Developer_counts_dict_1980[Developer] += 1
    else:
        Developer_counts_dict_1980[Developer] = 1
sorted_Developer_counts_1980 = sorted(Developer_counts_dict_1980.items(), key=lambda item: item[1], reverse=True)
for Developer, count in sorted_Developer_counts_1980:
    print(f"Developer_1980: {Developer}, Game Count: {count}")

Developer_counts_dict_1990 = {}
for Developer in df_1990['Developer']:
    if Developer in Developer_counts_dict_1990:
        Developer_counts_dict_1990[Developer] += 1
    else:
        Developer_counts_dict_1990[Developer] = 1
sorted_Developer_counts_1990 = sorted(Developer_counts_dict_1990.items(), key=lambda item: item[1], reverse=True)
for Developer, count in sorted_Developer_counts_1990:
    print(f"Developer_1990: {Developer}, Game Count: {count}")

Developer_counts_dict_2000 = {}
for Developer in df_2000['Developer']:
    if Developer in Developer_counts_dict_2000:
        Developer_counts_dict_2000[Developer] += 1
    else:
        Developer_counts_dict_2000[Developer] = 1
sorted_Developer_counts_2000 = sorted(Developer_counts_dict_2000.items(), key=lambda item: item[1], reverse=True)
for Developer, count in sorted_Developer_counts_2000:
    print(f"Developer_2000: {Developer}, Game Count: {count}")

Developer_counts_dict_2010 = {}
for Developer in df_2010['Developer']:
    if Developer in Developer_counts_dict_2010:
        Developer_counts_dict_2010[Developer] += 1
    else:
        Developer_counts_dict_2010[Developer] = 1
sorted_Developer_counts_2010 = sorted(Developer_counts_dict_2010.items(), key=lambda item: item[1], reverse=True)
for Developer, count in sorted_Developer_counts_2010:
    print(f"Developer_2010: {Developer}, Game Count: {count}")

Developer_counts_dict_2020 = {}
for Developer in df_2020['Developer']:
    if Developer in Developer_counts_dict_2020:
        Developer_counts_dict_2020[Developer] += 1
    else:
        Developer_counts_dict_2020[Developer] = 1
sorted_Developer_counts_2020 = sorted(Developer_counts_dict_2020.items(), key=lambda item: item[1], reverse=True)
for Developer, count in sorted_Developer_counts_2020:
    print(f"Developer_2020: {Developer}, Game Count: {count}")
```
## 4. Computing and displaying top 10 game developers for each decade (1970s, 1980s, 1990s, 2000s, 2010s, and 2020s) based on their average **critic scores**
* Grouping the data by **'Developer'**.
* Filtering out invalid critic scores.
* Calculating the average critic score for each developer.
* Sorting the developers by their average critic score in descending order.
* Printing out the top 10 developers for each decade.

```python
grouped_1970 = df_1970.groupby('Developer')['Critic Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each Developer
avg_scores_dict_1970 = {}

for index, row in grouped_1970.iterrows():
    # Filtering out invalid scores
    valid_scores_1970 = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_1970:
        avg_1970 = sum(valid_scores_1970) / len(valid_scores_1970)
        avg_scores_dict_1970[row['Developer']] = (avg_1970, len(valid_scores_1970))

# Sorting Developers by their average scores in descending order
sorted_Developers_1970 = sorted(avg_scores_dict_1970.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_1970[:10], 1):
  #11 to cover unknown
    print("1970")
    print(f"Developer: {Developer}")
    print("The average critic values for each Developer is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")

grouped_1980 = df_1980.groupby('Developer')['Critic Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each Developer
avg_scores_dict_1980 = {}

for index, row in grouped_1980.iterrows():
    # Filtering out invalid scores
    valid_scores_1980 = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_1980:
        avg_1980 = sum(valid_scores_1980) / len(valid_scores_1980)
        avg_scores_dict_1980[row['Developer']] = (avg_1980, len(valid_scores_1980))

# Sorting Developers by their average scores in descending order
sorted_Developers_1980 = sorted(avg_scores_dict_1980.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_1980[:10], 1):
    print("1980")
    print(f"Developer: {Developer}")
    print("The average critic values for each Developer is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")

grouped_1990 = df_1980.groupby('Developer')['Critic Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each Developer
avg_scores_dict_1990 = {}

for index, row in grouped_1990.iterrows():
    # Filtering out invalid scores
    valid_scores_1990 = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_1990:
        avg_1990 = sum(valid_scores_1990) / len(valid_scores_1990)
        avg_scores_dict_1990[row['Developer']] = (avg_1990, len(valid_scores_1990))

# Sorting Developers by their average scores in descending order
sorted_Developers_1990 = sorted(avg_scores_dict_1990.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_1990[:10], 1):
    print("1990")
    print(f"Developer: {Developer}")
    print("The average critic values for each Developer is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")

grouped_2000 = df_2000.groupby('Developer')['Critic Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each Developer
avg_scores_dict_2000 = {}

for index, row in grouped_2000.iterrows():
    # Filtering out invalid scores
    valid_scores_2000 = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_2000:
        avg_2000 = sum(valid_scores_2000) / len(valid_scores_2000)
        avg_scores_dict_2000[row['Developer']] = (avg_2000, len(valid_scores_2000))

# Sorting Developers by their average scores in descending order
sorted_Developers_2000 = sorted(avg_scores_dict_2000.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_2000[:10], 1):
    print("2000")
    print(f"Developer: {Developer}")
    print("The average critic values for each Developer is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")

grouped_2010 = df_2010.groupby('Developer')['Critic Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each Developer
avg_scores_dict_2010 = {}

for index, row in grouped_2010.iterrows():
    # Filtering out invalid scores
    valid_scores_2010 = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_2010:
        avg_2010 = sum(valid_scores_2010) / len(valid_scores_2010)
        avg_scores_dict_2010[row['Developer']] = (avg_2010, len(valid_scores_2010))

# Sorting Developers by their average scores in descending order
sorted_Developers_2010 = sorted(avg_scores_dict_2010.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_2010[:10], 1):
    print("2010")
    print(f"Developer: {Developer}")
    print("The average critic values for each Developer is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")

grouped_2020 = df_2020.groupby('Developer')['Critic Score'].apply(list).reset_index()

# Creating a dictionary to store average scores for each Developer
avg_scores_dict_2020 = {}

for index, row in grouped_2020.iterrows():
    # Filtering out invalid scores
    valid_scores_2020 = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_2020:
        avg_2020 = sum(valid_scores_2020) / len(valid_scores_2020)
        avg_scores_dict_2020[row['Developer']] = (avg_2020, len(valid_scores_2020))

# Sorting Developers by their average scores in descending order
sorted_Developers_2020 = sorted(avg_scores_dict_2020.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_2020[:10], 1):
  #11 to account for unknown
    print("2020")
    print(f"Developer: {Developer}")
    print("The average critic values for each Developer is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")
```
## 5. Computing and displaying top 10 game developers for each decade (1970s, 1980s, 1990s, 2000s, 2010s, and 2020s) based on their average **user scores**
Doing the same steps as the last time, but analyzing user scores.

```python
grouped_1970 = df_1970.groupby('Developer')['User Score'].apply(list).reset_index()

avg_scores_dict_1970 = {}

for index, row in grouped_1970.iterrows():
    valid_scores_1970 = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_1970:
        avg_1970 = sum(valid_scores_1970) / len(valid_scores_1970)
        avg_scores_dict_1970[row['Developer']] = (avg_1970, len(valid_scores_1970))

sorted_Developers_1970 = sorted(avg_scores_dict_1970.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_1970[:10], 1):
  #11 to cover unknown
    print("1970")
    print(f"Developer: {Developer}")
    print("The average user values for each Developer is:", round(avg, 2))
    print(f"Number of user scores: {count}")
    print("\n")

grouped_1980 = df_1980.groupby('Developer')['User Score'].apply(list).reset_index()

avg_scores_dict_1980 = {}

for index, row in grouped_1980.iterrows():
    valid_scores_1980 = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_1980:
        avg_1980 = sum(valid_scores_1980) / len(valid_scores_1980)
        avg_scores_dict_1980[row['Developer']] = (avg_1980, len(valid_scores_1980))

sorted_Developers_1980 = sorted(avg_scores_dict_1980.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_1980[:10], 1):
    print("1980")
    print(f"Developer: {Developer}")
    print("The average user values for each Developer is:", round(avg, 2))
    print(f"Number of user scores: {count}")
    print("\n")

grouped_1990 = df_1980.groupby('Developer')['User Score'].apply(list).reset_index()

avg_scores_dict_1990 = {}

for index, row in grouped_1990.iterrows():
    valid_scores_1990 = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_1990:
        avg_1990 = sum(valid_scores_1990) / len(valid_scores_1990)
        avg_scores_dict_1990[row['Developer']] = (avg_1990, len(valid_scores_1990))

sorted_Developers_1990 = sorted(avg_scores_dict_1990.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_1990[:10], 1):
    print("1990")
    print(f"Developer: {Developer}")
    print("The average user values for each Developer is:", round(avg, 2))
    print(f"Number of user scores: {count}")
    print("\n")

grouped_2000 = df_2000.groupby('Developer')['User Score'].apply(list).reset_index()

avg_scores_dict_2000 = {}

for index, row in grouped_2000.iterrows():
    valid_scores_2000 = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_2000:
        avg_2000 = sum(valid_scores_2000) / len(valid_scores_2000)
        avg_scores_dict_2000[row['Developer']] = (avg_2000, len(valid_scores_2000))

sorted_Developers_2000 = sorted(avg_scores_dict_2000.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_2000[:10], 1):
    print("2000")
    print(f"Developer: {Developer}")
    print("The average user values for each Developer is:", round(avg, 2))
    print(f"Number of user scores: {count}")
    print("\n")

grouped_2010 = df_2010.groupby('Developer')['User Score'].apply(list).reset_index()

avg_scores_dict_2010 = {}

for index, row in grouped_2010.iterrows():
    valid_scores_2010 = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_2010:
        avg_2010 = sum(valid_scores_2010) / len(valid_scores_2010)
        avg_scores_dict_2010[row['Developer']] = (avg_2010, len(valid_scores_2010))

sorted_Developers_2010 = sorted(avg_scores_dict_2010.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_2010[:10], 1):
    print("2010")
    print(f"Developer: {Developer}")
    print("The average user values for each Developer is:", round(avg, 2))
    print(f"Number of user scores: {count}")
    print("\n")

grouped_2020 = df_2020.groupby('Developer')['User Score'].apply(list).reset_index()

avg_scores_dict_2020 = {}

for index, row in grouped_2020.iterrows():
    valid_scores_2020 = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]

    if valid_scores_2020:
        avg_2020 = sum(valid_scores_2020) / len(valid_scores_2020)
        avg_scores_dict_2020[row['Developer']] = (avg_2020, len(valid_scores_2020))

sorted_Developers_2020 = sorted(avg_scores_dict_2020.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)

for i, (Developer, (avg, count)) in enumerate(sorted_Developers_2020[:10], 1):
  #11 to account for unknown
    print("2020")
    print(f"Developer: {Developer}")
    print("The average user values for each Developer is:", round(avg, 2))
    print(f"Number of user scores: {count}")
    print("\n")
```

# Finding the Top Differences between Critic Score and User Score of Developers for each decade
## 1. Filtering out rows where scores are valid
```python
valid_df_1970 = df_1970.dropna(subset=['Critic Score', 'User Score']).copy()
```
* Here, the code removes rows (games) where either the **Critic Score** or **User Score** is not available (NaN).
* **.copy()** creates a copy of the dataframe so that the original dataframe isn't affected by subsequent operations.

## 2. Calculating the absolute difference between Critic and User scores:
```python
valid_df_1970.loc[:, 'Difference'] = abs(valid_df_1970['Critic Score'] - valid_df_1970['User Score'])
```
* This line computes the absolute difference between the critic and user scores and stores it in a new column named 'Difference'.

## 3. Sorting by the difference and selecting top 10 rows
```python
top_10_diff_1970 = valid_df_1970.sort_values(by='Difference', ascending=False).head(10)
```
## 4. Displaying the developers with the maximum difference
* Code shows the example of print for 1970s decade
* This process repeats for each of the decades in the dataset.

```python
print("1970")
print(top_10_diff_1970[['Developer', 'Difference']])
```

# Calculating the Average Percentage of Japanese Sales
## Grouped the dataframe by Japan Sales and calculated total sales:
```python
grouped_1970 = df_1970.groupby('Japan Sales')['Total Sales'].apply(list).reset_index()
```
* This groups the dataframe by Japan Sales and computes a list of Total Sales for each unique value of Japan Sales.
## 2. Loop through the grouped dataframe to compute percentages
```python
for index, row in grouped_1970.iterrows():
    valid_scores_1970 = [game for game in row['Japan Sales'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]
    if valid_scores_1970:
        percentage_1970 = (grouped_1970['Japan Sales'] /grouped_1970['Total Sales']) * 100
        average_percentage = percentage_1970.mean()
        print("The average percentage sales for Japan is : ", average_percentage)
```
* This loop goes through each row of the grouped dataframe.
* We filtered out invalid sales values.
* For valid values, it computes the percentage of Japanese sales with respect to total sales.
* Finally, it prints out the average percentage of sales for Japan.



# 5 decades maximum total sales analysis
## 1. For calculating this we had to group the developer columns and the total sales column and find the sum of the total sales. After doing this, we had to sort it in descending order.
```python
#1970 sales
df_1970['Total Sales'] = df_1970['Total Sales'].str[:-1].astype(float)
grouped_sales = df_1970.groupby('Developer')['Total Sales'].sum()
sorted_sales = grouped_sales.sort_values(ascending=False)
print(sorted_sales.head(10))
```
## 2. Console track data analysis
We separated the console name and how many games were created from each console.
#1970 example
```python
grouped = df_1970.groupby('Console')['Game'].apply(list).reset_index()
for index, row in grouped.iterrows():
    print(f"Console: {row['Console']}")
    for game in row['Game']:
        print(f" - {game}")
    print("\n")
```

## 3. We found the average value for each console and sorted by descending order **FOR CRITIC Scores**
#1970 example
```python
grouped_1970 = df_1970.groupby('Console')['Critic Score'].apply(list).reset_index()


# Creating a dictionary to store average scores for each Console
avg_scores_dict_1970 = {}


for index, row in grouped_1970.iterrows():
    # Filtering out invalid scores
    valid_scores_1970 = [game for game in row['Critic Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]


    if valid_scores_1970:
        avg_1970 = sum(valid_scores_1970) / len(valid_scores_1970)
        avg_scores_dict_1970[row['Console']] = (avg_1970, len(valid_scores_1970))


# Sorting Consoles by their average scores in descending order
sorted_Consoles_1970 = sorted(avg_scores_dict_1970.items(), key=lambda x: (x[1][0], x[1][1]), reverse=True)


for i, (Console, (avg, count)) in enumerate(sorted_Consoles_1970[:10], 1):
  #11 to cover unknown
    print("1970")
    print(f"Console: {Console}")
    print("The average critic values for each Console is:", round(avg, 2))
    print(f"Number of critic scores: {count}")
    print("\n")
```
## 4. The same process was done with the **USER Values**
```python
grouped_1970 = df_1970.groupby('Console')['User Score'].apply(list).reset_index()


avg_scores_dict_1970 = {}


for index, row in grouped_1970.iterrows():
    valid_scores_1970 = [game for game in row['User Score'] if game != '' and pd.notna(game) and isinstance(game, (int, float))]
```
## 5. We found the difference between average user values and critic values
#1970 example
```python
# Filtering the dataframe for rows where both 'Critic Score' and 'User Score' are not NaN
valid_df_1970 = df_1970.dropna(subset=['Critic Score', 'User Score']).copy()


# Calculating the absolute difference between 'Critic Score' and 'User Score'
valid_df_1970.loc[:, 'Difference'] = abs(valid_df_1970['Critic Score'] - valid_df_1970['User Score'])


# Sorting the dataframe by the 'Difference' column in descending order and getting the top 10 rows
top_10_diff_1970 = valid_df_1970.sort_values(by='Difference', ascending=False).head(10)


# Displaying the Console(s) with the maximum difference
print("1970")
print(top_10_diff_1970[['Console', 'Difference']])
```

