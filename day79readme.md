#seaborn
## Learning Goals
* Create a **choropleth** to display data on a map.
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

```python

# * Calculate the average prize share of the winners on a year by year basis.
yearly_share = df_data.groupby(by='year').share_pct.mean()

rolling_yearly_share = yearly_share.rolling(window=5).mean()

# Show a tick mark on the x-axis for every 5 years from 1900 to 2020. (Hint: you'll need to use NumPy).
plt.xticks(ticks=np.arange(1900, 2021, 5),
           rotation=45)
ax1 = plt.gca()
ax1.set_ylabel("Prizes Awarded", color='dodgerblue')
ax1.set_xlim(1900, 2020)

ax2 = ax1.twinx()
ax2.set_ylabel("Rolling Average Prize Split Percentage", rotation=90, color='teal')
ax2.set_ylim(40, 100)
# Invert the Y-axis
ax2.invert_yaxis()




ax2.plot(rolling_yearly_share.index, rolling_yearly_share.values, color='teal')
# Using Matplotlib superimpose the rolling average on a scatter plot.
# Use the named colours to draw the data points in dogerblue while the rolling average is coloured in crimson.

## Add a second axis and plot the rolling average of the prize share on it

ax1.plot(rolling_prizes_per_year.index, rolling_prizes_per_year.values, color='crimson')


plt.show()
```

#### Challenge 2

Investigate if more prizes are shared than before.

1. Calculate the average prize share of the winners on a year by year basis.
    `yearly_share = df_data.groupby(by='year').share_pct.mean()`
	solution: `yearly_avg_share = df_data.groupby(by='year').agg({'share_pct': pd.Series.mean})`
 `share_moving_average = yearly_avg_share.rolling(window=5).mean()`
 Using agg results in a named column with np.float64 type values. My original solution produces 
2.  Calculate the 5 year rolling average of the percentage share.
   `rolling_yearly_share = yearly_share.rolling(window=5).mean()` 
3. Copy-paste the cell from the chart you created above.
    
4. Modify the code to add a secondary axis to your Matplotlib chart.
    
5. Plot the rolling average of the prize share on this chart.
    
6. See if you can invert the secondary y-axis to make the relationship even more clear.

#### My Solution
```python
# Show a tick mark on the x-axis for every 5 years from 1900 to 2020. (Hint: you'll need to use NumPy).
plt.xticks(ticks=np.arange(1900, 2021, 5),
           rotation=45)
ax1 = plt.gca()
ax1.scatter(prizes_per_year.index, prizes_per_year.values, color='dodgerblue')
ax1.plot(rolling_prizes_per_year.index, rolling_prizes_per_year.values, color='crimson')

ax1.set_ylabel("Prizes Awarded", color='dodgerblue')
ax1.set_xlim(1900, 2020)

ax2 = ax1.twinx()  # Create a second axis.
ax2.plot(rolling_yearly_share.index, rolling_yearly_share.values, color='teal')
ax2.set_ylabel("Rolling Average Prize Split Percentage", rotation=90, color='teal')
ax2.set_ylim(40, 100)
# Invert the Y-axis
ax2.invert_yaxis()
# Using Matplotlib superimpose the rolling average on a scatter plot.
# Use the named colours to draw the data points in dogerblue while the rolling average is coloured in crimson.

## Add a second axis and plot the rolling average of the prize share on it
plt.show()
```

### Solution

```python
plt.figure(figsize=(16, 8), dpi=200)
plt.title('Number of Novel Prizes Awarded per Year', fontsize=18)
plt.yticks(fontsize=14)
plt.xticks(ticks=np.arange(1900, 2021, step=5),
		  fontsize=14,
		  rotation=45
		  )

ax1=plt.gca()
ax2 = ax1.twinx()
ax1.set_xlim(1900, 2020)

ax1.scatter(x=prize_per_year.index,
		   y=prize_per_year.values,
		   c='dodgerblue',
		   alpha=0.7,
		   s=100
		   )

ax1.plot(prize_per_year.index,
		moving_average.values,
		c='crimson',
		linewidth=3,
		)

# Adding a prize share plot on second axis.

ax2.plot(prize_per_year.index,
		share_moving_average.values,
		c='grey',
		linewidth=3,
		)
# Invert the y-axis of ax2
ax2.invert_yaxis()
plt.show()
```

## choropleth Map and Countries with the Most Prizes

Which countries actually get the most prizes? And which categories are those prizes in?

### Top 20 Country Ranking

**top20_countries**

| country | prizes |
| --- | --- |
| some country | prizes won |

Which is the best column for country? `birth_country`, `birth_country_current`, or `organization_country`?

organization_country is NaN for individuals
birth_country contains countries that no longer exist
birth_country_current is the best choice


next chalenge:

*Hint*: Take a two-step approach. The first step is grouping the data by country and category. Then you can create a DataFrame that looks something like this:

<img src=https://i.imgur.com/VKjzKa1.png width=450>

## Solution

```python

top_countries = df_data.groupby(by=['birth_country_current'], as_index=False)\
						.agg({'prize'}: pd.Series.count)
top20_countries = top_countries.sort_values(by='prize', inplace=True)[-20:]

# Make a bar chart

h_bar = px.bar(x=top20_countries.prize,
			  y=top20_countries.birth_country_current,
			  orientation='h',
			  color=top20_countries.prize,
			  color_continuous_scale='Viridis',
			  title='Top 20 Countries by Number of Prizes')
h_bar.update_layout(xaxis_title='Number of Prizes',
				   yaxis_title='Country',
				   coloraxis_showscale=False)
h_bar.show()

##### MINE #####

top20_bar = px.bar(top20_countries[:20], x='prize', y='birth_country_current',
                  )
top20_bar.update_layout(
title='Top 20 Countries by Nobel Prize Winners',
    yaxis={'categoryorder': 'total ascending'},
    yaxis_title='Country (Current)',
    xaxis_title='Prizes Won',
)

## Choropleth Map

df_countries = df_data.groupby(by=['birth_country_current', 'ISO'],
							  as_index=False).agg({'prize': pd.Series.count})
df_countries.sort_values('prize', ascending=False)

## We can use the ISO country codes for the locations parameters on the choropleth.

world_map = px.choroplet(df_countries,
						locations='ISO',
						color='prize',
						hover_name='birth_country_current',
						color_continous_scale=px.colors.sequential.matter)
world_map.update_layout(coloraxis_showscale=True)
world_map.show()


#### MY ABSURDLY CONVOLUTED SOLUTION ####

df_data.ISO
test = top20_countries.copy()
test = pd.merge(top20_countries, df_data[['birth_country_current', 'ISO']], left_on='birth_country_current', right_on='birth_country_current').drop_duplicates()
test.index = top20_countries.index
top20_countries.iso_alpha = test.ISO

choropleth_map = px.choropleth(top20_countries,
                                locations='iso_alpha',
                                 color='prize',
                                 hover_name='birth_country_current',
                                 color_continuous_scale = 'matter'
                                ).show()

## Category Breakdown by Country

cat_country = df_data.groupby(by=['birth_country_current', 'category'],
							 as_index=False).agg({'prize': pd.Series.count})
cat_country.sort_values(by='prize', ascending=False, inplace=True)

# Merge cat_country with top20_countries so we get a column with the total numver of prizes too

merged_df = pd.merge(cat_country, top20_countries, on='birth_country_current')

merged_df.columns = ['birthday_country_current', 'category', 'cat_prize', 'total_prize']
merged_df.sort_values(by='total_prize', inplace=True)


## No we create a new bar chart using color to represent category prizes

cat_cntry_bar = px.bar(x=merged_df.cat_prize,
					  y=merged_df.birth_country_current,
					  color=merged_df.category,
					  orientation='h',
					  title='Top 20 countries by Number of Prizes and Category')
cat_cntry_bar.upate_layout(xaxis_title='Number of Prizes',
						  yaxis_title='Country')

#####MINE#####
new_df = df_data.groupby(by=['birth_country_current', 'category'], as_index=False).agg({'prize': pd.Series.count})
total_country_prizes = new_df.groupby('birth_country_current', as_index=False).agg({'prize': pd.Series.count})
# new_df.rename(columns={'prize': 'cat_prize'}, inplace=True)
# new_df.birth_country_current.nunique()
new_df

cat_country_df = pd.merge(left=new_df, right=top20_countries, on='birth_country_current')
# cat_country_df.drop(['total_prize', 'iso_alpha'], axis=1, inplace=True)
# cat_country_df.rename(columns={"prize": "total_prize"}, inplace=True)

cat_country_df.sort_values(by='total_prize', ascending=False, inplace=True)

top20_bar = px.bar(cat_country_df[:50], x='cat_prize', y='birth_country_current',
                   color='category'
                  )
top20_bar.update_layout(
title='Top 20 Countries by Nobel Prize Winners',
    yaxis={'categoryorder': 'total ascending'},
#     coloraxis={'order': 'total descending'},
    yaxis_title='Country (Current)',
    xaxis_title='Prizes Won',
)


## Country Prizes over Time
# Count the number of prizes by country by year

prize_by_year = df_data.groupby(by=['birth_country_current', 'year'], as_index=False).count()
prize_by_year = prize_by_year.sort_values('year')[['year', 'birth_country_current', 'prize']]
# Create a series that has a cumulative sum for the number of prizes won

cumulative_prizes = prize_by_year.groupby(by=['birth_country_current', 'year']).sum().groupby(level=[0]).cumsum()
cumulative_prizes.reset_index(inplace=True)

l_chart = px.line(cumulative_prixes,
				 x='year',
				 y='prize',
				 color='birth_country_current',
				 hover_name='birth_country_current')
l_chat.update_layout(xaxis_title='Year',
					yaxis_title='Number of Prizes Won')
l_chart.show()
####MINE####
######################IT'S FUCKIN CRAZY########################
df_data.head(20)
# Every country needs equal number of data points
# US has 281
# France has 57
# Sum/count all prizes awarded to country in year and make that the new value for year index


COUNTRY_QUERY = "United States of America"
COUNTRY_CONDITION = df_data.birth_country_current == COUNTRY_QUERY

df_data.loc[COUNTRY_CONDITION, "cumulative_wins"]
len(df_data.loc[COUNTRY_CONDITION, "cumulative_wins"])

df_data[df_data.birth_country_current == "United States of America"][df_data.cumulative_wins == df_data.cumulative_wins.cummax()].year.min()

# US wins eclipsed other countries in 1950

COUNTRY_TO_GRAPH = "France"

df_data.groupby(by=['birth_country_current']).agg({'year': pd.Series.nunique})
wins_by_year = df_data[['year', 'birth_country_current', 'cumulative_wins']]
# print(wins_by_year.dropna().reset_index())
px.line(
       wins_by_year[wins_by_year.birth_country_current == "France"].cumulative_wins)
index_is_year = wins_by_year.copy()
index_is_year.index = index_is_year.year
index_is_year.drop('year', axis=1, inplace=True)
print(index_is_year)
list_of_countries = index_is_year.birth_country_current.unique()
index_is_year.groupby('birth_country_current', as_index=False).size()
px.line(index_is_year[index_is_year.birth_country_current == COUNTRY_TO_GRAPH].cumulative_wins).show()
#        index_is_year[index_is_year.birth_country_current == "Germany"].cumulative_wins)

pd.pivot_table(index_is_year, values="cumulative_wins", index=index_is_year.index, columns="birth_country_current").France

index_is_year[index_is_year.birth_country_current == COUNTRY_TO_GRAPH].cumulative_wins.info()

year_country_wins = df_data[['year', 'birth_country_current', 'prize']].copy()
year_country_wins.loc[:, "did_win"] = 1  # Creating a column that just =1 for wins
year_country_wins.drop(index="Prizes_Won_Up_To_Year", inplace=True)
yearly_wins_all_countries = year_country_wins.groupby(['year', 'birth_country_current'], as_index=False).agg({'did_win': pd.Series.sum})
print(yearly_wins_all_countries)
# us_cum_wins = year_country_wins.loc[year_country_wins.birth_country_current == "United States of America", "did_win"].cumsum().reset_index().drop(columns="index")
# fr_cum_wins = year_country_wins.loc[year_country_wins.birth_country_current == "France", "did_win"].cumsum().reset_index().drop(columns="index")
 


# us_cum_wins
# fr_cum_wins.index = us_cum_wins.index

pd.pivot_table(year_country_wins, values="did_win", columns="birth_country_current", index="year")

# wins per year
# You can chain .agg() to get running tally
# FRESHING YEARLY_WINS DATAFRAME
def starting_country_win_data(country: str):
    yearly_wins_all_countries.index = np.int64(yearly_wins_all_countries.year)
    yearly_wins_all_countries.drop(columns=['year'], inplace=True)
    condition = yearly_wins_all_countries.birth_country_current == country

    return yearly_wins_all_countries[condition]
#     return yearly_wins = yearly_wins_all_countries[yearly_wins_all_countries.birth_country_current == country].groupby('year').agg({'did_win': pd.Series.count})  #.agg({'did_win': pd.Series.cumsum})
france_data = starting_country_win_data("France").copy()
france_data.rename(columns={'did_win': 'France'}, inplace=True)
france_data
yearly_wins_all_countries["year"] = yearly_wins_all_countries.index
yearly_wins_all_countries

yearly_wins_all_countries.groupby(['year']).agg({'did_win': pd.Series.cumsum})
# Adding a running tally column to yearly_wins_all_countries
# yearly_wins_all_countries["tally"] = yearly_wins_all_countries.groupby(['year','birth_country_current']).agg({'did_win': pd.Series.cumsum})
# yearly_wins_all_countries

yearly_wins.index = np.int64(yearly_wins.index)

def add_no_win_years_data(country: str, df: pd.DataFrame):
    before_wins = True
    for year in range(1901, 2021):
        if year in df.index:
            before_wins = False
            continue
        elif before_wins:
            df.loc[year, country] = 0
        else:
            df.loc[year, country] = df.loc[year - 1, country]
add_no_win_years_data(country="USA", df=yearly_wins)
yearly_wins.sort_index(inplace=True)

fr_cum_wins = add_no_win_years_data(country='france'.title(), df=france_data)
fr_tally = france_data.drop(columns='birth_country_current').sort_index().cumsum().copy()
yearly_wins

# yearly_wins["France"] = fr_tally["France"].copy()
# yearly_wins["year"] = yearly_wins.index
# yearly_wins.reset_index(inplace=True)
# yearly_wins.drop("index", axis=1)

# px.line(pivoted_yearly_wins, x=yearly_wins.columns, y='USA')
px.data.gapminder().query("continent == 'Oceania'")

test = df_data.groupby(['birth_country_current', 'year'], as_index=False).cumsum(numeric_only=True).dropna()
test

df_data_clean = df_data.dropna(subset=['cumulative_wins'])
px.line(df_data_clean, x='year', y='cumulative_wins', color='birth_country_current')


```

## Top Research Organizations

```python
# Reset the dataframe
df_data = pd.read_csv('nobel_prize_data.csv')
orgs_df = df_data.dropna(subset=['organization_name']).groupby('organization_name', as_index=False).agg({'prize': pd.Series.count})

top20_orgs_df = orgs_df.sort_values('prize')[-20:]

top20orgs_bar = px.bar(top20_orgs_df,
      x='prize',
      y='organization_name',
       color='prize',
      orientation='h',
      color_continuous_scale='Rdbu',
      title="Top 20 Research Instutions by Number of Prizes",
                      hover_name='organization_name')

top20orgs_bar.update_layout(xaxis_title="Number of Prizes",
                           yaxis_title="Institution",
                           coloraxis_showscale=False)
                   
```
![[Pasted image 20231129193447.png]]


## Top Research Organizations

solution:
```python
top20_orgs = df_data.organization_name.value_counts()[:20]
top20_orgs.sort_values(ascending=True, inplace=True)
# Mine
orgs_df = df_data.dropna(subset=['organization_name']).groupby('organization_name', as_index=False).agg({'prize': pd.Series.count})

top20_orgs_df = orgs_df.sort_values('prize')[-20:]

top20orgs_bar = px.bar(top20_orgs_df,
      x='prize',
      y='organization_name',
       color='prize',
      orientation='h',
      color_continuous_scale='Rdbu',
      title="Top 20 Research Instutions by Number of Prizes",
                      hover_name='organization_name')

top20orgs_bar.update_layout(xaxis_title="Number of Prizes",
                           yaxis_title="Institution",
                           coloraxis_showscale=False)

```

## Research Cities
## Date stuff

`Series.dt.date returns only the Y / M / D portion of a timestamp`

```python
age_df = df_data[["year", "birth_date"]].copy()
df_data.laureate_type
# # Convert the birth_data column to a timestamp
age_df["birth_date_timestamp"] = pd.to_datetime(age_df["birth_date"])
# Convert Timestamp to a datetime object
age_df["birth_year"]  = age_df["birth_date_timestamp"].dt.date
# Acess the datetime objet's year attribute
age_df["birth_year"] = [dt.year for dt in age_df.birth_year]
# Create the winning_age column. 
age_df["winning_age"] = age_df["year"] - age_df["birth_year"]

df_data["winning_age"] = [int(n) for n in age_df["winning_age"].fillna(0).copy()]
```

### Youngest an doldest

```python
youngest = df_data[df_data.laureate_type == 'Individual']\
                .loc[df_data[df_data.laureate_type == 'Individual'].winning_age.idxmin()]
young_name = youngest["full_name"]
young_prize = youngest["prize"]

oldest = df_data[df_data.laureate_type == 'Individual']\
                .loc[df_data[df_data.laureate_type == 'Individual'].winning_age.idxmax()]
old_name = oldest["full_name"]
old_prize = oldest["prize"]
print(f"The youngest laureate is {young_name}. They won {young_prize}.")
print(f"The oldest is {old_name}. They won {old_prize}")
```

### Mean age of winners:
```python
mean_age = df_data.describe().loc["mean", "winning_age"]
int(round(mean_age, 0))
```

## What the fuck is a histogram?

```python
mean_age = df_data.describe().loc["mean", "winning_age"]
int(round(mean_age, 0))
age_value_count = df_data.winning_age.value_counts().sort_index()

# Seaborn histogram
# import seaborn as sns, for some reason
age_hist = sns.histplot(data=age_value_count,
                           x=age_value_count.index,
                           y=age_value_count.values,
                           bins=5)
```
![[example_histogram_day79.png]]

#### with 20 bins
![[day79_hist_20_bins.png]]
This shows the range of the number of winners within 5 - year windows.  Not sure what the different shades represent. [[Learn More]](https://seaborn.pydata.org/generated/seaborn.histplot.html)

## Descriptive Statistics

???

## Age of Winner Over Time
Make a [regplot](https://seaborn.pydata.org/generated/seaborn.regplot.html?highlight=regplot#seaborn.regplot)

Create a dataframe with the year and the average age of the winners (of that year, or up until that year? Not sure how the .mean() works in this case.)

```python
avg_age_over_time = df_data.copy().groupby('year', as_index=False)["winning_age"].mean()
avg_age_over_time.rename(columns={'winning_age': 'mean_winning_age'},
                         inplace=True)

 age_reg = sns.regplot(data=avg_age_over_time,
 x = "year",
 y="mean_winning_age")
```
![[day79_age_regmodel.png]]
 # What would this predict for a 2020 winner? 
* Add a grid
`with sns.axes_style({"axes.grid": True}):`

