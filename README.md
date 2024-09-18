# Phase_1_Project_File
# MICROSOFT MOVIE STUDIO PROJECT ANALYSIS

## Introduction

###  Project Overview
Microsoft has decided to venture into movie industry and wants to launch its own movie studio.To successfully do this, we need to look at the trends and factors contributing to the success of other movie studios.

### 2. Goals
1. Analyze movie data to uncover insights for decision making
2. Leverage data from several box offices
3. Identify patterns and attributes that correlate with box office performance

### 3. Data Sources
For purpose of this project we used data from:
1. Box Office Mojo - Provided information on box office revenue.
2. IMDB - Detailed information on movie basics and ratings.

### 4. Methodology
1. Data Collection - data from Box Office Mojo and IMDB used each providing different insights into movie performance.
2. Data Cleaning, Standardization and Integration - we cleaned the datasets, standardized and intergrated them ensuring that our analysis maintains the consistency and reliability required.
3. Exploratory Data Analysis (EDA) - we performed extensive EDA to uncover correlations, trends, and actionable insights.

### 5. Expected Results
By the end of this analysis, we will address:
1. Information on most profitable genres
2. Insights on genres, budgeting strategies and audince preferences
3. Conclusions to guide Microsoft's new venture into movie industry based on data analysis and clear visualizations.

## Data Preparation
### 1. Extracting data and perform initial exploration of datatsets

Import relevant modules

``` python
import csv
import pandas as pd
import sqlite3
import os


```
Loading CSV Files
```python
box_office_data = pd.read_csv("bom.movie_gross.csv.gz")


```
Loading SQLite Database
``` python
conn = sqlite3.connect("im.db")


```
performing initial data explorations
```python
box_office_data.head()

```
```python
table_list_query = "SELECT name FROM sqlite_master WHERE type='table';"
tables = pd.read_sql_query(table_list_query, conn)

print("Available tables:", tables)

```
```python
movie_ratings = pd.read_sql("""SELECT * FROM movie_ratings;""", conn)
movie_basics = pd.read_sql("""SELECT * FROM movie_basics;""", conn)

conn.close()

print(movie_ratings.head())
print(movie_basics.head())

```
### 2. Data Aggregation and Cleaning
1. Combine movie_basics and movie_ratings, then merge output with with the box_office_data
2. Handle missing values - drop unnecessary columns and fill missing values in key columns
3. Standardize genre labels

merge IMDB data

```python
imdb_data = pd.merge(movie_basics, movie_ratings, on='movie_id', how='left')

print(imdb_data.head())

```
Merge imdb merging output to the box_office_data

```python
merged_data = pd.merge(box_office_data, imdb_data, left_on='title', right_on='primary_title', how='left')
print(merged_data)

```
Check missing values in merged data

```python
print(merged_data.isna().sum())

```
Drop columns not necessary to our analysis
```python
merged_data.drop(columns=['original_title', 'start_year', 'numvotes'], inplace=True)

```
#### Handle Missing values in key columns

1. filling missing studio values with 'Unknown'
```python
merged_data['studio'].fillna('Unknown', inplace=True)

```

2. filling missing domestic gross and foreign gross with '0'
```python
merged_data['domestic_gross'].fillna(0, inplace=True)
merged_data['foreign_gross'].fillna(0, inplace=True)

```

3. finding median runtime and replacing missing runtimes
```python
median_runtime = merged_data['runtime_minutes'].median()
merged_data['runtime_minutes'].fillna(median_runtime, inplace=True)

```

4. filling missing genres with 'Unknown'
```python
merged_data['genres'].fillna('Unknown', inplace=True)

```

5. finding mean rating and replace to missing average ratings
```python
mean_rating = merged_data['averagerating'].mean()
merged_data['averagerating'].fillna(mean_rating, inplace=True)

```

6. dropping rows that do not have movie id
```python
merged_data.dropna(subset=['movie_id'], inplace=True)

```

7. check if there are further missing values
```python
print(merged_data.isna().sum())

```
Check if data is clean before proceeding to analysis
```python
print(merged_data.head())

```

#### Standardize genre column
```python

def clean_and_explode(df, genre_column):
    df[genre_column] = df[genre_column].str.split(',')
    df = df.explode(genre_column).reset_index(drop=True)
    return df

merged_data = clean_and_explode(merged_data, 'genres')

merged_data['averagerating'] = merged_data['averagerating'].fillna(merged_data['averagerating'].mean())

print(merged_data.head())

```
## Data Analysis
We have our data and we need to perform analysis to identifying the best-performing genre. Ensure that financial data(domestic and foreign gross) are numeric values for easier analysis.

1. Ensuring that revenue data is captured as int or float for analysis
```python

merged_data['domestic_gross'] = pd.to_numeric(merged_data['domestic_gross'], errors='coerce').fillna(0).astype(int)
merged_data['foreign_gross'] = pd.to_numeric(merged_data['foreign_gross'], errors='coerce').fillna(0).astype(int)

```

2. Aggregated Domestic and Foreign Revenue by Genre
```python

genre_revenue = merged_data.groupby('genres')[['domestic_gross', 'foreign_gross', 'averagerating']].agg({
    'domestic_gross': ['sum', 'mean'],
    'foreign_gross': ['sum', 'mean'],
    'averagerating': 'mean'
}).reset_index()

```

2. rename main columns for easier identity
```python

genre_revenue.columns = ['genre', 'total_domestic_gross', 'average_domestic_gross', 'total_foreign_gross', 'average_foreign_gross', 'average_rating']

```

3. Sort by total domestic gross revenue
```python

genre_revenue = genre_revenue.sort_values(by='total_domestic_gross', ascending=False)

print(genre_revenue)

```
## Data Visualization
Import plotting modules

```python
import seaborn as sns
import matplotlib.pyplot as plt

```
### 1. Best-rated genre

``` python

plt.figure(figsize=(14, 7))
sns.barplot(x='genre', y='average_rating', data=genre_revenue)
plt.title('Average Rating by Genre')
plt.xlabel('Genre')
plt.ylabel('Average Rating')
plt.xticks(rotation=45)
plt.show()

```
### 2. Genre income from the domestic market

```python

plt.figure(figsize=(14, 7))
sns.barplot(x='genre', y='total_domestic_gross', data=genre_revenue)
plt.title('Total Domestic Gross by Genre')
plt.xlabel('Genre')
plt.ylabel('Total Domestic Gross (Mil)')
plt.xticks(rotation=45)
plt.show()

```
### 3. Genre income from the international market

```python
plt.figure(figsize=(14, 7))
sns.barplot(x='genre', y='total_foreign_gross', data=genre_revenue)
plt.title('Total Foreign Gross by Genre')
plt.xlabel('Genre')
plt.ylabel('Total Foreign Gross (mil)')
plt.xticks(rotation=45)
plt.show()

```

### 4. Correlation between ratings and revenue by genre

```python
plt.figure(figsize=(10, 6))
sns.scatterplot(x='average_rating', y='total_domestic_gross', data=genre_revenue, label='Domestic Gross')
sns.scatterplot(x='average_rating', y='total_foreign_gross', data=genre_revenue, label='Foreign Gross')
plt.title('Correlation between Ratings and Revenue by Genre')
plt.xlabel('Average Rating')
plt.ylabel('Total Gross (Mil)')
plt.legend()
plt.show()

```
### 5. Comparison of revenue generated in the domestic and foreign markets

```python
plt.figure(figsize=(12, 6))
plt.plot(genre_revenue['genre'], genre_revenue['total_domestic_gross'], marker='o', label='Total Domestic Gross')
plt.plot(genre_revenue['genre'], genre_revenue['total_foreign_gross'], marker='s', label='Total Foreign Gross')
plt.title('Total Domestic vs Total Foreign Gross by Genre')
plt.xlabel('Genre')
plt.ylabel('Gross Revenue (Mil)')
plt.xticks(rotation=45)
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

```
## Insights and Business Recommendations
From the analysis, this is what we recommend to Microsoft:

### 1. Focus on top-performing genres
Action, Adventure and Animation genres have shown strong performance on the locan and international markets.

### 2. Invest in the highly rated genres
Microsoft should build on genres where higher ratings strongly correlate with it success. Animation, Biography and Drama genres indicates that positive reviews significantly boosts its revenue.

### 3. Target global market
Microsoft should consider investing in the global audience. Our analysis indicates that almost every genre does well in the foreign market compared to the domestic market.
