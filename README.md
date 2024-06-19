# covid_19-analysis
#Exploratory Data Analysis and Visualization of COVID-19 Data
# Import necessary libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Load the dataset
url = 'https://covid.ourworldindata.org/data/owid-covid-data.csv'
df = pd.read_csv(url)

# Inspect the dataset
print(df.info())
print(df.describe())
print(df.head())

# Data Cleaning
df = df.dropna(subset=['total_cases', 'total_deaths'])  # Remove rows with missing critical values
df['date'] = pd.to_datetime(df['date'])  # Convert date column to datetime
df = df.fillna(0)  # Fill remaining missing values with 0

# Data Wrangling
# Check if 'total_recoveries' column exists
if 'total_recoveries' in df.columns:
    df['active_cases'] = df['total_cases'] - df['total_deaths'] - df['total_recoveries']
    df['recovery_rate'] = (df['total_recoveries'] / df['total_cases']) * 100
else:
    print("Warning: 'total_recoveries' column not found. Active cases and recovery rate cannot be calculated.")

df['fatality_rate'] = (df['total_deaths'] / df['total_cases']) * 100 # This calculation can still be performed

# Filter data for specific countries (e.g., USA, India, Brazil)
countries = ['United States', 'India', 'Brazil']
df_countries = df[df['location'].isin(countries)]

# Exploratory Data Analysis
# Line plot for trends over time
plt.figure(figsize=(14, 7))
for country in countries:
    country_data = df_countries[df_countries['location'] == country]
    plt.plot(country_data['date'], country_data['total_cases'], label=country)
plt.title('COVID-19 Total Cases Over Time')
plt.xlabel('Date')
plt.ylabel('Total Cases')
plt.legend()
plt.show()

# Bar plot for total cases, deaths, and vaccinations
plt.figure(figsize=(10, 5))
total_cases = df_countries.groupby('location')['total_cases'].max()
total_deaths = df_countries.groupby('location')['total_deaths'].max()
total_vaccinations = df_countries.groupby('location')['total_vaccinations'].max()
data = pd.DataFrame(
    {
    'Total Cases': total_cases,
    'Total Deaths': total_deaths,
    'Total Vaccinations': total_vaccinations
    }
)
data.plot(kind='bar')
plt.title('COVID-19 Cases, Deaths, and Vaccinations by Country')
plt.ylabel('Count')
plt.show()

# Heatmap for correlation
plt.figure(figsize=(10, 5))
# Remove 'recovery_rate' from the list of columns if it's still not present
columns_for_heatmap = ['total_cases', 'total_deaths', 'total_vaccinations', 'tests_per_case', 'fatality_rate']
if 'recovery_rate' in df_countries.columns:
    columns_for_heatmap.append('recovery_rate')
sns.heatmap(df_countries[columns_for_heatmap].corr(), annot=True)
plt.title('Correlation Matrix')
plt.show()

# Scatter plot for testing rate vs. case count
plt.figure(figsize=(10, 5))
sns.scatterplot(data=df_countries, x='tests_per_case', y='total_cases', hue='location')
plt.title('Testing Rate vs. Total Cases')
plt.show()
