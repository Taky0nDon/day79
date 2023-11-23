#seaborn
## Learning Goals
* Create a **Chloropleth** to display data on a map.
* Create bar charts showing different segments of the data with plotly.
* Create Sunburst charts with plotly.
* Use Seaborn's `.lmplot()` and show best-fit llines across multiple categories using the `row`, `hue`, and `lowess` parameters.
* Understand how a differen tpicture emerges when looking at the same data in different ways (e.g., box plots vs a time series analysis).
* See the distribution of our data and visualize descriptive statistics with the help of a histogram in Seaborn.

## Fun pip fact
`pip show <package>` outputs information about the package, including the version.

## Challenge 1

### Preliminary data exploration.

- What is the shape of `df_data`? How many rows and columns?
- What are the column names and what kind of data is inside of them?
- In which year was the Nobel prize first awarded?
- Which year is the latest year included in the dataset?

```python
rows, cols = df_data.shape
print(f"df_data has {rows} rows and {cols} columns")
col_names: str = ", ".join([name for name in df_data.columns])
print(f"The columns are: {col_names}")
print(f"The Nobel Prize was first awared in {df_data.year.min()}")
print(f"the most recent year in the dataset is {df_data.year.max()}")
```

>df_data has 962 rows and 16 columns
>The columns are: year, category, prize, motivation, prize_share, laureate_type, full_name, birth_date, birth_city, birth_country, birth_country_current, sex, organization_name, organization_city, organization_country, ISO
>The Nobel Prize was first awared in 1901
>the most recent year in the dataset is 2020

#### Challenge 2
#### Which columns have NaN values?

`[column for column in df_data.isna().any().index if df_data[column].isna().values.any() == True]  # columns containing NaN values`
	\['motivation',
	 'birth_date',
	 'birth_city',
	 'birth_country',
	 'birth_country_current',
	 'sex',
	 'organization_name',
	 'organization_city',
	 'organization_country',
	 'ISO']

- Are there any duplicate values in the dataset?
- Are there NaN values in the dataset?
- Which columns tend to have NaN values?
- How many NaN values are there per column?
- Why do these columns have NaN values?

```python
df_data.duplicated().values.any()  # False, no duplicates
df_data.isna().values.any()  # True, dos contain NaN values
nan_containing_columns = [column for column in df_data.isna().any().index if df_data[column].isna().values.any() == True]  # columns containing NaN values

df_data.isna().sum().sum() / len(nan_containing_columns)  # 102.3 NaN values per column

df_data[df_data.isna().any(axis=1)]  # These columns have naN values if there was no involvement of an organization in accepting the prize, or if no
# value was given for that column.
```
#### how to display rows containing NaN values

##### filtering on the NaN values in a given column:
`df_data.loc[df_data[column].isna()]`

`df_data[df_data.isna().any(axis=1)`
The boolean determination is with respect to columns. renders the rows where any column contains a NaN value.
## Challenge 3

- Convert the `birth_date` column to Pandas `Datetime` objects
    `df_data.birth_date = pd.to_datetime(df_data.birth_date)`
- Add a Column called `share_pct` which has the laureates' share as a percentage in the form of a floating-point number.
```python
share_ints = [string.split('/') for string in df_data.prize_share.values]
for thing in share_ints:
    for i, value in enumerate(thing):
        thing[i] = int(value)
share_ints[0]
df_data['share_pct'] = [(pair[0] / pair[1]) * 100 for pair in share_ints]
df_data
```

##### solution

```python
separated_values = df_data.prize_share.str.split('/', expand=True)
numerator = pd.to_numeric(separated_values[0])
denominator = pd.to_numeric(separated_values[1])
df_data['share_pct'] = numerator / denominator
```

## Questions

1. How likely is a nobel prize to be awarded to an organization?
2. Which category of prize has been awarded most frequently?
3. Which countries have the most nobel prize winners?

## Solutions

1. Creating a donut chart
```
biology = df_data.sex.value_counts()
fig = px.pie(labels=biology.index,
values=biology.values,
names=biology.index,
hole=0.4)
fig.update_traces(textposition='inside',
textfont_size=15,
textinfo='percent'
)
```

2. First 3 women to win

```
df_data[df_data.sex == 'Female'].sort_values(by='year',
ascending=True)[:3]
```

3. Repeat Winners
```
is_winner = df_data.duplicated(subset=['full_name'], keep=False)
mutiple_winners = df_data[is_winner]
print(f'There are {multiple_winners.full_name.nunique()} entities that have won more than one nobel prize.')
```

4. Prizes per category

```
prizes_per_category = df_data.category.value_counts()
fig = px.bar(x=prizes_per_category.index,
y=prizes_per_category.values,
color=prizes_per_category.values,
color_continuous_scale='Aggrnyl',
title='Number of Prizes Awarded Per Category')

fig.update_layout(xaxis_title='Nobel Prize Category',
yaxis_title='Number of Prizes',
coloraxis_showscale=False)
```

5. The Economics Prize

```
df_data[df_data.category == 'Economics'].sort_values(by='year')[:3]
```

6. Male and female winners by category

```
cat_men_women = df_data.groupby(['category', 'sex'],
as_index=False).agg({'prize': pd.Series.count})
cat_men_women.sort_values(by='prize',
ascending=False,
inplace=True)

v_bar_split = px.bar(x=cat_men_women.category,
y=cat_men_women.prize,
color=cat_men_women.sex)
v_bar_split.show()
```

## Multi-axis charts

see [[day74readme]]

### Challenge 1

Show the trend in rate of prize rewards with respect to time. Are more prizes given out now than when the award was first introduced?

* Count the number of prizes awarded every year.
```python
prizes_per_year = df_data[df_data.year <= 2020].year.value_counts()
#SOLUTION
prize_per_year = df_data.groupby(by='year').count().prize
```
* Create a 5 year rolling average of the number of prizes (see [[day73readme]])
```python
rolling_prizes_per_year = prizes_per_year.sort_index().rolling(window=5, min_periods=0).mean()
#SOLUTION
rolling = prize_per_year.rolling(window=5).mean()
```
* Using Matplotlib superimpose the rolling average on a scatter plot
```python
prizes_per_year_copy = prizes_per_year.copy()
prizes_per_year_copy.index = new_years

# Using Matplotlib superimpose the rolling average on a scatter plot.
# Use the named colours to draw the data points in dogerblue while the rolling average is coloured in crimson.

scatter = plt.scatter(prizes_per_year_copy.index, prizes_per_year_copy.values, color='dodgerblue')
plt.plot(rolling_copy.index, rolling_copy.values, color='crimson')

# Show a tick mark on the x-axis for every 5 years from 1900 to 2020. (Hint: you'll need to use NumPy).
new_years = [np.datetime64(str(year), 'D') for year in prizes_per_year.index]  # Forces every year into a Y-01-01 format
every_5_years = mpl_dates.YearLocator(base=5)

ax1 = plt.gca()
ax1.xaxis.set_major_locator(every_5_years)
ax1.set_xlim(min(new_years), max(new_years))

plt.xticks(rotation=315)
plt.show()
#SOLUTION
# Generate an array

np.arange(1900, 2021, step=5)
# Use our array to define the charts ticks

plt.xticks(ticks=np.arange(1900, 2021, step=5))
```
* Show a tick mark on the x-axis for every 5 years from 1900 to 2020
```python
# Need to use NumPy

new_years = sorted([np.datetime64(str(year), 'D') for year in prizes_per_year.index])  # Forces every year into a Y-01-01 format

rolling_copy = rolling_prizes_per_year.copy()
rolling_copy.index = new_years
rolling_copy
#SOLUTION

```
* Use `named colours` to draw the data points in `dogerblue` while the rolling average is colored in `crimson`
* Did the first and second world wars have an impact on the number of prizes being given out?
* What could be the reason for the trend in the chart?