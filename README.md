General notes:
look into movies that are less expensive. This might be a way to limit capital risk if Microsoft doesn't want to go super hard into the investment. 
look at distribution of the top grossing files. Maybe give them a distribution of files to go for, show which ones will have pretty reliable income and which ones are the longshots but have huge payoff potential. 

imports os and glob
use pd.merge

<h1>
    Mircosoft: Taping into the movie industry
</h1>
<h5>
    Prepared by: Michael Mahoney
</h5>

* Student pace:  part time
* Scheduled project review date/time: 
* Instructor name: James Irving
* Blog post URL:

<h1>
    Introduction:
</h1>
<p>
This notebook marks an initial exploratory investigation of the movie industry on behalf of Microsoft. It contains the technical analysis and source code for which I base my recomendations. A copy of this notebook will be provided in the requested format for future use and/or investigation should Microsoft endeavor to insert themselves in the move market.
</p>
<p>
    There is a liberal use of markdown in this notebook for the purpose of elusidating my thought process during the investigation. In general, comments in the code are reserved for technical python notes and not the methodology of the investigation.
</p>



```python
#dependancies in this notebook

from IPython.display import display, Markdown, Latex
import pandas as pd
import datetime
import numpy as np
import matplotlib.pyplot as plt
import requests
import json
import seaborn as sns
plt.style.use('fivethirtyeight')

```

The following cell contains pandas options for how much info is displayed for DataFrames. We have set the columns to display all and the rows are unaltered. If you want to see all entries, uncomment the second line. 


```python
pd.set_option('display.max_columns', None)
# pd.set_option('display.max_rows', None)
```

<h1>
    Cleaning the data:
</h1>
<p>
    As with all data analysis, we begin by understanding and cleaning the data. 
</p>
<br>


```python
#import the files from the local repo
dfBomGross = pd.read_csv('zippedData/bom.movie_gross.csv.gz')
dfimdbName = pd.read_csv('zippedData/imdb.name.basics.csv.gz')
dfimdbTitleBasics = pd.read_csv('zippedData/imdb.title.basics.csv.gz')
dfimdbTitleCrew = pd.read_csv('zippedData/imdb.title.crew.csv.gz')
dfimdbTitlePrincipals = pd.read_csv('zippedData/imdb.title.principals.csv.gz')
dfimdbTitleRatings = pd.read_csv('zippedData/imdb.title.ratings.csv.gz')
dfRtMovie = pd.read_csv('zippedData/rt.movie_info.tsv.gz',sep='\t')
dfRtReviews = pd.read_csv('zippedData/rt.reviews.tsv.gz',sep='\t', encoding='latin_1')
dfTmbd = pd.read_csv('zippedData/tmdb.movies.csv.gz')
dfTn = pd.read_csv('zippedData/tn.movie_budgets.csv.gz')

#We add column headers to this file which was missing them
dfimdbTitleAkas = pd.read_csv('zippedData/imdb.title.akas.csv.gz')
columns = list(dfimdbTitleAkas.columns)
columns[0] = 'tconst'
dfimdbTitleAkas.columns = columns

#Display first row of all DataFrames to get a first look at the data structures
listOfDfs = [dfBomGross, dfimdbName, dfimdbTitleAkas, dfimdbTitleBasics, dfimdbTitleCrew, dfimdbTitlePrincipals, dfimdbTitleRatings, dfRtMovie, dfRtReviews, dfTmbd, dfTn]
for x in listOfDfs:
    display(x.head(1))
print('\n')
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>studio</th>
      <th>domestic_gross</th>
      <th>foreign_gross</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Toy Story 3</td>
      <td>BV</td>
      <td>415000000.0</td>
      <td>652000000</td>
      <td>2010</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>nm0061671</td>
      <td>Mary Ellen Bauder</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miscellaneous,production_manager,producer</td>
      <td>tt0837562,tt2398241,tt0844471,tt0118553</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering</th>
      <th>title</th>
      <th>region</th>
      <th>language</th>
      <th>types</th>
      <th>attributes</th>
      <th>is_original_title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0369610</td>
      <td>10</td>
      <td>Джурасик свят</td>
      <td>BG</td>
      <td>bg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.0</td>
      <td>Action,Crime,Drama</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>directors</th>
      <th>writers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0285252</td>
      <td>nm0899854</td>
      <td>nm0899854</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering</th>
      <th>nconst</th>
      <th>category</th>
      <th>job</th>
      <th>characters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0111414</td>
      <td>1</td>
      <td>nm0246005</td>
      <td>actor</td>
      <td>NaN</td>
      <td>["The Man"]</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>averagerating</th>
      <th>numvotes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt10356526</td>
      <td>8.3</td>
      <td>31</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>synopsis</th>
      <th>rating</th>
      <th>genre</th>
      <th>director</th>
      <th>writer</th>
      <th>theater_date</th>
      <th>dvd_date</th>
      <th>currency</th>
      <th>box_office</th>
      <th>runtime</th>
      <th>studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>This gritty, fast-paced, and innovative police...</td>
      <td>R</td>
      <td>Action and Adventure|Classics|Drama</td>
      <td>William Friedkin</td>
      <td>Ernest Tidyman</td>
      <td>Oct 9, 1971</td>
      <td>Sep 25, 2001</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>104 minutes</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>review</th>
      <th>rating</th>
      <th>fresh</th>
      <th>critic</th>
      <th>top_critic</th>
      <th>publisher</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3</td>
      <td>A distinctly gallows take on contemporary fina...</td>
      <td>3/5</td>
      <td>fresh</td>
      <td>PJ Nabarro</td>
      <td>0</td>
      <td>Patrick Nabarro</td>
      <td>November 10, 2018</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>genre_ids</th>
      <th>id</th>
      <th>original_language</th>
      <th>original_title</th>
      <th>popularity</th>
      <th>release_date</th>
      <th>title</th>
      <th>vote_average</th>
      <th>vote_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>[12, 14, 10751]</td>
      <td>12444</td>
      <td>en</td>
      <td>Harry Potter and the Deathly Hallows: Part 1</td>
      <td>33.533</td>
      <td>2010-11-19</td>
      <td>Harry Potter and the Deathly Hallows: Part 1</td>
      <td>7.7</td>
      <td>10788</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
  </tbody>
</table>
</div>


    
    
    

<br>
<p>
    We see from the above cell that the various parts of our working data are in disarray in terms for formatting and usefulness. Thinking about our ultimate goal, every business centers around the basic premise of profitability (or operational sustainability for those non-profits out there). To that end, any real discussion of how to best enter the movie industry begins with money. Two of our data sources have this information. 
</p>
<br>
<br>


```python
dfTn.info()
dfBomGross.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5782 entries, 0 to 5781
    Data columns (total 6 columns):
     #   Column             Non-Null Count  Dtype 
    ---  ------             --------------  ----- 
     0   id                 5782 non-null   int64 
     1   release_date       5782 non-null   object
     2   movie              5782 non-null   object
     3   production_budget  5782 non-null   object
     4   domestic_gross     5782 non-null   object
     5   worldwide_gross    5782 non-null   object
    dtypes: int64(1), object(5)
    memory usage: 271.2+ KB
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 3387 entries, 0 to 3386
    Data columns (total 5 columns):
     #   Column          Non-Null Count  Dtype  
    ---  ------          --------------  -----  
     0   title           3387 non-null   object 
     1   studio          3382 non-null   object 
     2   domestic_gross  3359 non-null   float64
     3   foreign_gross   2037 non-null   object 
     4   year            3387 non-null   int64  
    dtypes: float64(1), int64(1), object(3)
    memory usage: 132.4+ KB
    

<br>
<br>
We do a quick check to see if the information overlaps and how well.
<br>
<br>


```python
df2 = dfTn.merge(dfBomGross, how = 'left', left_on='movie', right_on = 'title')
df2.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 5782 entries, 0 to 5781
    Data columns (total 11 columns):
     #   Column             Non-Null Count  Dtype  
    ---  ------             --------------  -----  
     0   id                 5782 non-null   int64  
     1   release_date       5782 non-null   object 
     2   movie              5782 non-null   object 
     3   production_budget  5782 non-null   object 
     4   domestic_gross_x   5782 non-null   object 
     5   worldwide_gross    5782 non-null   object 
     6   title              1247 non-null   object 
     7   studio             1246 non-null   object 
     8   domestic_gross_y   1245 non-null   float64
     9   foreign_gross      1086 non-null   object 
     10  year               1247 non-null   float64
    dtypes: float64(2), int64(1), object(8)
    memory usage: 542.1+ KB
    

<br>
<br>
    From the info method we see that essentially all the information in dfBomGross is encapsulated by dfTn, therefore we can abandon the join and use dfTn directly. Not to mention dfTn has substantially more accuracy in terms of the numbers themselves. 
   <br>
   <br>


```python
df2 = dfTn.copy()
df2.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>$330,600,000</td>
      <td>$459,005,868</td>
      <td>$1,403,013,963</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>$317,000,000</td>
      <td>$620,181,382</td>
      <td>$1,316,721,747</td>
    </tr>
  </tbody>
</table>
</div>



<br>
dfTn does, however, have a substantial amount of formatting issues which we fix in the following cell. 


```python

# convert various columns object data to int data for analysis
df2['domestic_gross'] = [x.replace(',','').replace('$','') for x in df2['domestic_gross']]
df2['domestic_gross'] = df2['domestic_gross'].astype('int64')
df2['worldwide_gross'] = [x.replace(',','').replace('$','') for x in df2['worldwide_gross']]
df2['worldwide_gross'] =df2['worldwide_gross'].astype('int64')
df2['production_budget'] = [x.replace(',','').replace('$','') for x in df2['production_budget']]
df2['production_budget'] =df2['production_budget'].astype('int64')

# Add a new column for profit 
df2['profit'] = (df2['worldwide_gross'] - df2['production_budget'])
df2['profit'] = df2['profit'].astype('float64')
#Change release date to a useable datetime format and add some more columns for easier access
df2['release_date'] = pd.to_datetime(df2['release_date'])
df2['release_year'] = pd.DatetimeIndex(df2['release_date']).year
df2['release_month'] = pd.DatetimeIndex(df2['release_date']).month
df2['release_day'] = pd.DatetimeIndex(df2['release_date']).weekday
# df2 = df2.merge(dfInfo, how = 'left', left_on = 'movie', right_on = 'primary_title')
df2.head(1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>profit</th>
      <th>release_year</th>
      <th>release_month</th>
      <th>release_day</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2009-12-18</td>
      <td>Avatar</td>
      <td>425000000</td>
      <td>760507625</td>
      <td>2776345279</td>
      <td>2.351345e+09</td>
      <td>2009</td>
      <td>12</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




<br>
<p>
    Looking at this structure, it's clear that joining tables with only financial information isn't going to give much insight as to why movies fiscally perform the way they do. For the sake of saving time later, I'm going to combine all of the IMDB info here and use the names of the movies we do have income information on to sort through the IMDB list which is much more complete in terms of information.
</p>


```python
#check data frame method keys. Use x.columns
# With the exception of dfimdbName, all IMDB dataframes have a common primary key which we use to join them

#The following code changes all 'tconst' columns into a common data type
listOfImdbDfs = [dfimdbTitleAkas,dfimdbTitleBasics,dfimdbTitleCrew,dfimdbTitlePrincipals,dfimdbTitleRatings]
for x in listOfImdbDfs:
    if('tconst' in x.keys()):
        x['tconst'] = x['tconst'].astype('str')

# We define a function to make the joining a little easier
def joinThings(df1, df2):
    df = df1.merge(df2,how='left',left_on='tconst', right_on='tconst')
    return df

# This is still messy and could use some love
df = joinThings(joinThings(joinThings(joinThings(dfimdbTitleAkas,dfimdbTitleBasics),dfimdbTitleCrew),dfimdbTitlePrincipals),dfimdbTitleRatings)
display(df.head(1), df.info())
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2841276 entries, 0 to 2841275
    Data columns (total 22 columns):
     #   Column             Dtype  
    ---  ------             -----  
     0   tconst             object 
     1   ordering_x         int64  
     2   title              object 
     3   region             object 
     4   language           object 
     5   types              object 
     6   attributes         object 
     7   is_original_title  float64
     8   primary_title      object 
     9   original_title     object 
     10  start_year         int64  
     11  runtime_minutes    float64
     12  genres             object 
     13  directors          object 
     14  writers            object 
     15  ordering_y         float64
     16  nconst             object 
     17  category           object 
     18  job                object 
     19  characters         object 
     20  averagerating      float64
     21  numvotes           float64
    dtypes: float64(5), int64(2), object(15)
    memory usage: 498.6+ MB
    


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tconst</th>
      <th>ordering_x</th>
      <th>title</th>
      <th>region</th>
      <th>language</th>
      <th>types</th>
      <th>attributes</th>
      <th>is_original_title</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>directors</th>
      <th>writers</th>
      <th>ordering_y</th>
      <th>nconst</th>
      <th>category</th>
      <th>job</th>
      <th>characters</th>
      <th>averagerating</th>
      <th>numvotes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0369610</td>
      <td>10</td>
      <td>Джурасик свят</td>
      <td>BG</td>
      <td>bg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Jurassic World</td>
      <td>Jurassic World</td>
      <td>2015</td>
      <td>124.0</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>nm1119880</td>
      <td>nm0415425,nm0798646,nm1119880,nm2081046,nm0000341</td>
      <td>10.0</td>
      <td>nm0189777</td>
      <td>producer</td>
      <td>producer</td>
      <td>NaN</td>
      <td>7.0</td>
      <td>539338.0</td>
    </tr>
  </tbody>
</table>
</div>



    None


<br>
<p>
    This information is much more juicy in terms of disecting movie performace. However, there's more work to be done eliminating bad data so we will have something manageable to merge into our financial information. Let's begin by taking a look at which columns or rows we should think about dropping. 
</p>
<br>


```python
#percentage of null values per column
df.isna().sum()/len(df)*100
```




    tconst                0.000000
    ordering_x            0.000000
    title                 0.000000
    region               16.217010
    language             86.635406
    types                45.353954
    attributes           95.215143
    is_original_title     0.000880
    primary_title         0.000000
    original_title        0.000458
    start_year            0.000000
    runtime_minutes       7.938300
    genres                0.864154
    directors             0.494531
    writers               8.058492
    ordering_y            0.034914
    nconst                0.034914
    category              0.034914
    job                  74.427933
    characters           61.141156
    averagerating        14.711594
    numvotes             14.711594
    dtype: float64



<br>
<p>
    This informs the following cell in which we dump either incomplete or eteraneous data from df. I want to note at this point that I will not be removing averagerating despite the high percentage of missing data. This is becuase the average rating appears to be at the heart of commercial success and we have enough information to be able to lose .
</p>
<br>


```python
df.drop(columns = ['language', 'types', 'attributes', 'job', 'characters','category', 'ordering_y', 'ordering_x', 'nconst', 'is_original_title', 'region', 'title', 'original_title'], inplace = True)
```

<br>
<br>
<p>
    A subtle outcome that I want to bring attention to is a standard, but significant, consequence of collapsing dimensional data. Thousands of the rows in the combined IMDB dataframe have been reduced to duplicates which will be shown and removed in the following cell.
</p>
<br>


```python
# We display the shape of the dataframe before and after dropping columns
display(df.shape)
df['duplicated'] = df.duplicated()
df = df.drop(df.loc[df['duplicated']].index)
display(df.shape)
```


    (2841276, 9)



    (122302, 10)



<br>
<p>
     Because the very large df dataframe is going to be merged into the much smaller df2 dataframe, it might seem acedemic that we've reduced the df dataframe by 100,000 entries. But this is very important for the joining process which would otherwise be much more likely to have critical and hard to detect errors if the duplicates remained. With those removed, we can go ahead and join this into our financial data
</p>
<br>


```python
listOfNames = [x for x in df2.movie.unique()]
def test(x):
    if x in listOfNames:
        return True
    else:
        return False
df['inDf2'] = df['primary_title'].isin(listOfNames)
dfInfo = df.loc[df['inDf2']]
dfFinal = df2.merge(dfInfo, how = 'left', left_on = 'movie', right_on = 'primary_title')

```


<h1>
    Question 1
</h1>

* What Type of movies make the most money


```python
display(np.corrcoef(dfFinal['production_budget'], dfFinal['profit'])[0][1])

sns.jointplot(x = 'production_budget', y ='profit', data = dfFinal, height = 12, kind = 'reg' ,joint_kws = {'line_kws' : {'color':'red'}});
ax = plt.gca();
ax.set_title("Profit Vs Production Budget");
```


    0.6060881813278928



![png](output_27_1.png)


<h2>
    Answer for Question 1:
</h2>
<p>
    Notice the positive correlation of <strong>0.606</strong>. This is highly significant and good evidence that a higher production budget leads to a higher profit. 
</p>
<br>
<br>


<h1>
    Question 2
</h1>
<br>
What are the most profitable Genres?
<br>
<br>


```python
#first we are going to re-add some of the info we dropped for the average rating analysis. 
listOfNames = [x for x in df2.movie.unique()]
def test(x):
    if x in listOfNames:
        return True
    else:
        return False
df['inDf2'] = df['primary_title'].isin(listOfNames)
dfInfo = df.loc[df['inDf2']]
dfFinal = df2.merge(dfInfo, how = 'left', left_on = 'movie', right_on = 'primary_title')
```

We are doing something non-standard in the following cells. Because the genre column contains multiple genres per cell
 we are going to use prime numbers in order to retain this information in an accesible way.


```python
# we are doing something non-standard. Because the genre column contains multiple genres per cell
# we are going to use prime numbers in order to retain this information in an accesible way. 
# We first create a list of primes to assign to our different genres
def isPrime(num):
    test = [num % x != 0 for x in range(2,num)]
    if all(test):
        return num
primes = []   
for x in range(2, 1000):
    if isPrime(x):
        primes.append(isPrime(x))

```


```python
# find all the unique genres in the genres column
dfFinal['genres'] = dfFinal['genres'].astype('str')

genresList = []
for x in dfFinal['genres']:
    temp = x.lower().strip().split(',')
    for i in temp:
        if i in genresList:
            continue
        else:
            genresList.append(i)
            
# Create a dictionary that assigns primes to each of the unique genres
genresDict = {genresList[i]:primes[i] for i in range(len(genresList))}
# Split the genres column up in order to assign primes
dfFinal['genres'] = [x.strip().lower().split(',') for x in dfFinal['genres']]

# Fuction that multiplies all the primes together
def productFunc(array):
    runningTotal = 1
    for x in array:
        runningTotal = runningTotal*genresDict[x]
    return runningTotal

# Create the genre code column and add some true false columns for slicing later
dfFinal['genresCode'] = [productFunc(x) for x in dfFinal['genres']]  
dfFinal.head()

# http://jonathansoma.com/lede/data-studio/classes/small-multiples/long-explanation-of-using-plt-subplots-to-create-small-multiples/


```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>profit</th>
      <th>release_year</th>
      <th>release_month</th>
      <th>release_day</th>
      <th>tconst</th>
      <th>primary_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>directors</th>
      <th>writers</th>
      <th>averagerating</th>
      <th>numvotes</th>
      <th>duplicated</th>
      <th>inDf2</th>
      <th>genresCode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2009-12-18</td>
      <td>Avatar</td>
      <td>425000000</td>
      <td>760507625</td>
      <td>2776345279</td>
      <td>2.351345e+09</td>
      <td>2009</td>
      <td>12</td>
      <td>4</td>
      <td>tt1775309</td>
      <td>Avatar</td>
      <td>2011.0</td>
      <td>93.0</td>
      <td>[horror]</td>
      <td>nm3786927</td>
      <td>nm2179863,nm4392664</td>
      <td>6.1</td>
      <td>43.0</td>
      <td>False</td>
      <td>True</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2011-05-20</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000</td>
      <td>241063875</td>
      <td>1045663875</td>
      <td>6.350639e+08</td>
      <td>2011</td>
      <td>5</td>
      <td>4</td>
      <td>tt1298650</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>2011.0</td>
      <td>136.0</td>
      <td>[action, adventure, fantasy]</td>
      <td>nm0551128</td>
      <td>nm0254645,nm0744429,nm0064181,nm0938684,nm0694627</td>
      <td>6.6</td>
      <td>447624.0</td>
      <td>False</td>
      <td>True</td>
      <td>105</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2019-06-07</td>
      <td>Dark Phoenix</td>
      <td>350000000</td>
      <td>42762350</td>
      <td>149762350</td>
      <td>-2.002376e+08</td>
      <td>2019</td>
      <td>6</td>
      <td>4</td>
      <td>tt6565702</td>
      <td>Dark Phoenix</td>
      <td>2019.0</td>
      <td>113.0</td>
      <td>[action, adventure, sci-fi]</td>
      <td>nm1334526</td>
      <td>nm0126208,nm1079208,nm1079211,nm1334526,nm0456...</td>
      <td>6.0</td>
      <td>24451.0</td>
      <td>False</td>
      <td>True</td>
      <td>165</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2015-05-01</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000</td>
      <td>459005868</td>
      <td>1403013963</td>
      <td>1.072414e+09</td>
      <td>2015</td>
      <td>5</td>
      <td>4</td>
      <td>tt2395427</td>
      <td>Avengers: Age of Ultron</td>
      <td>2015.0</td>
      <td>141.0</td>
      <td>[action, adventure, sci-fi]</td>
      <td>nm0923736</td>
      <td>nm0923736,nm0498278,nm0456158,nm0800209,nm4160687</td>
      <td>7.3</td>
      <td>665594.0</td>
      <td>False</td>
      <td>True</td>
      <td>165</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>2017-12-15</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>317000000</td>
      <td>620181382</td>
      <td>1316721747</td>
      <td>9.997217e+08</td>
      <td>2017</td>
      <td>12</td>
      <td>4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[nan]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>13</td>
    </tr>
  </tbody>
</table>
</div>




```python
def createAThing(df):
    plotnumber = 1
    fig = plt.figure(figsize=(15,10))
    x= sorted(list(genresDict.keys()), reverse = False)
    y =[dfFinal.loc[dfFinal['genresCode'] % genresDict[x] == 0]['profit'].mean() for x in x]
    ax = sns.barplot(x, y, ax = plt.subplot(1,1, plotnumber));
    ax.set_title('Average Movie Profit Per Genre')
    ax.set_xlabel('Genre')
    ax.set_ylabel('Profit')
    ax.set_xticklabels(x, rotation= 60, fontdict={'horizontalalignment':'right'});
    plotnumber = plotnumber + 1
createAThing(dfFinal)
```


![png](output_34_0.png)


<h2>
    Answer for Question 2:
</h2>
<br>
<p>
    There appear to be several genres that are more commercialy viable, namely: Action, Adventure, Animation, fantasy, musical and Sci-fi genres.
<br>
<h3>
    In Depth Analysis for Question 2
</h3>
<br>


```python
# Defeine a function that create all out plots based on genre


def createAThing(df):
    fig = plt.figure(figsize=(50,50))
    plotnumber = 1
    for i in genresDict.keys():
        x = df.loc[df['genresCode'] % genresDict[i] == 0]['production_budget'];
        y = df.loc[df['genresCode'] % genresDict[i] == 0]['profit'];
        z = df.loc[df['genresCode'] % genresDict[i] == 0]['averagerating']
        
        ax = sns.scatterplot(x, y, ax = plt.subplot(6,4, plotnumber), hue = z);
        ax.set_title(f'{i.title()} Movies Profit Vs Rating')
        ax.set_xlabel('Production Budget')
        ax.set_ylabel('Profit')
        ax.legend(np.corrcoef(x,y)[1])
        
        plotnumber = plotnumber +1
createAThing(dfFinal)
```

    C:\ProgramData\Anaconda3\lib\site-packages\numpy\lib\function_base.py:2526: RuntimeWarning: Degrees of freedom <= 0 for slice
      c = cov(x, y, rowvar)
    C:\ProgramData\Anaconda3\lib\site-packages\numpy\lib\function_base.py:2455: RuntimeWarning: divide by zero encountered in true_divide
      c *= np.true_divide(1, fact)
    C:\ProgramData\Anaconda3\lib\site-packages\numpy\lib\function_base.py:2455: RuntimeWarning: invalid value encountered in multiply
      c *= np.true_divide(1, fact)
    


![png](output_36_1.png)


<br>
<h1>
    Question 3:
</h1>
<br>
<p>
    Who are the most profitable directors and writers?
</p>
<br>
<br>


```python
#get the names of the directors and writers
dfimdbName.loc[301124,'primary_name'] = 'James Cameron'
dfimdbName.loc[452518,'primary_name'] = 'James Cameron'
dfimdbName['isTopDirector'] = dfimdbName['nconst'].isin(dfFinal['directors'])
temp = dfimdbName.loc[dfimdbName['isTopDirector']]
dfFinal2 = dfFinal.merge(temp, how='left', left_on = 'directors', right_on = 'nconst')


```


```python
def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (15,10))
    x = df.groupby('primary_name').mean().sort_values('profit', ascending = False).index[:50]
    y = df.groupby('primary_name').mean().sort_values('profit', ascending = False)['profit'][:50]
    ax = sns.barplot(x = x, y = y, ax = plt.subplot(1,1,plotnumber), palette='Blues_d')
    ax.set_title('Top 50 Directors Per Average Movie Profit')
    ax.set_xlabel('Director Name')
    ax.set_ylabel('Profit')
    ax.set_xticklabels(x, rotation= 60, fontdict={'horizontalalignment':'right'});
    plotnumber = plotnumber +1

createANewThing(dfFinal2)
```


![png](output_39_0.png)



```python
# Get the top writers
dfimdbName['isTopWriter'] = dfimdbName['nconst'].isin(dfFinal['writers'])
temp = dfimdbName.loc[dfimdbName['isTopWriter']]
dfFinal3 = dfFinal.merge(temp, how='left', left_on = 'writers', right_on = 'nconst')


```


```python
def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (15,10))
    x = df.loc[df['isTopWriter'] == True].groupby('primary_name').mean().sort_values('profit', ascending = False).index[:50]
    y = df.loc[df['isTopWriter'] == True].groupby('primary_name').mean().sort_values('profit', ascending = False)['profit'][:50]
    ax = sns.barplot(x = x, y = y, ax = plt.subplot(1,1,plotnumber),palette='Reds_d')
    ax.set_title('Top 50 Writers Per Average Movie Profit')
    ax.set_xlabel('Writer Name')
    ax.set_ylabel('Profit')
    ax.set_xticklabels(x, rotation= 60, fontdict={'horizontalalignment':'right'});
    plotnumber = plotnumber +1

createANewThing(dfFinal3)
```


![png](output_41_0.png)


<br>
<h2>
    Answer for Question 3:
</h2>
<p>
    The above graphs give a short list of both writers and directors who have have had vast commercial success. By averaging the profits accross their movies, we cut down on fluke victories. Every name in the visuals should be considered, with special attention given to those with existing report.  
</p>
<br>
<br>

<h3>
    In Depth Analysis for Question 3
</h3>


```python
def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (60,60))
    fig.suptitle('Top Profitable Directors By Genre',fontsize = 120)
    for i in genresDict.keys():        
        x = df.loc[df['genresCode'] % genresDict[i] == 0].groupby('primary_name').mean().sort_values('profit', ascending = False).index[:50]
        y = df.loc[df['genresCode'] % genresDict[i] == 0].groupby('primary_name').mean().sort_values('profit', ascending = False)['profit'][:50]
        ax = sns.barplot(x = x, y = y, ax = plt.subplot(6,4,plotnumber), palette='Blues_d')
        ax.set_title(f'Top 50 {i.title()} Directors', fontdict = {'fontsize':36})
        ax.set_xlabel('Directors Name', fontdict = {'size' : 36})
        ax.set_ylabel('Average Profit', fontdict = {'size' : 36})
        ax.set_xticklabels(x, rotation= 60, fontdict={'horizontalalignment':'right'});
        plotnumber = plotnumber +1
        plt.tight_layout()

createANewThing(dfFinal2)

```



![png](output_44_1.png)


<br>
<p>
    This is a nice working list of the successful directors in each genres. Note that we averaged the profit per director, so in this sense, there is a possiblity some of the directors listed here are one hit wonders and do not have repeated mass commercial success. But, now that we have a short list, we have a manageable amount of follow up research. Lets move on to the writers
</p>
<br>
<br>


```python

dfimdbName['isTopWriter'] = dfimdbName['nconst'].isin(dfFinal['writers'])
temp = dfimdbName.loc[dfimdbName['isTopWriter']]
dfFinal3 = dfFinal.merge(temp, how='left', left_on = 'writers', right_on = 'nconst')

# we break out the top 50 grossing movies and their associated writers in each genre
plotnumber = 1
def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (60,60))
    fig.suptitle('Top Profitable Writers By Genre',fontsize = 120)
    for i in genresDict.keys():        
        x = df.loc[df['genresCode'] % genresDict[i] == 0].groupby('primary_name').mean().sort_values('profit', ascending = False).index[:50]
        y = df.loc[df['genresCode'] % genresDict[i] == 0].groupby('primary_name').mean().sort_values('profit', ascending = False)['profit'][:50]
        ax = sns.barplot(x = x, y = y, ax = plt.subplot(6,4,plotnumber),palette='Reds_d')
        ax.set_title(f'Top 50 {i.title()} Writers',fontdict = {'size' : 36})
        ax.set_xlabel('Writer Name', fontdict = {'size' : 36})
        ax.set_ylabel('Average Profit', fontdict = {'size' : 36})
        ax.set_xticklabels(x, rotation= 60, fontdict={'horizontalalignment':'right'});
        plotnumber = plotnumber +1
        plt.tight_layout()
createANewThing(dfFinal3)

```



![png](output_46_1.png)


<br>
<p>
    Here we note the same possibility of having one hit wonder writers. 
</p>
<br>
<br>

<h1>
    General Knowledge:
</h1>
<p>
    The following information seeks to give context to more overarching dynamics within the movie industry. They should be considered when approaching a major project but are not necessarily the most important factors. 
</p>

<h2>
    Movie profitability over time
</h2>
<p>
    The following graph is a rough summary of the movie industries profitability broken down by year
</p>


```python
dfFinal.sort_values('release_year',ascending = False, inplace = True)
fig,ax = plt.subplots(figsize = (20,10),
                      ncols = 1, nrows = 1)
x = dfFinal['release_year'].loc[dfFinal['release_year'] >= 1980]
y = dfFinal['profit'].loc[dfFinal['release_year'] >= 1980]
ax = sns.boxplot(x = x, y = y);
ax.set_title('Movie Profitability Over Time')
ax.set_xlabel('Release Year', fontdict = {'size' : 20})
ax.set_ylabel('Average Profit', fontdict = {'size' : 20});
ax.set_xticklabels(sorted(x.unique()), rotation =45, fontdict={'horizontalalignment':'center'});

```


![png](output_50_0.png)


<br>
<br>
<p>
    The general trend appears to be mostly flat overall. In more recent years, there have been many more block busters as represented in the higher 75% as well as the number of high grossing outliers in the past decade. It's worth pointing out that at the time of this analysis COVID-19 has decimated the movie industy going into 2020. While the bulk of this notebook address the historical trands of the movie industry, <strong>it is highly recomended that Microsoft does not invest substantially in the industry until the long term effects of COVID-19 are made less opaque.</strong>
</p>
<br>

<h2>
    Movie Ratings
</h2>
<p>
    There are several awards related to overal movie accredidation. It should be noted that if prestigue is the overal goal of the program then alternative summary statistics should be explored. The follow graph shows that overal move quality ratings are substantially worse predictors of commercial success than movie budget.
</p>


```python
x = dfFinal['averagerating']
y = dfFinal['profit']

display(np.corrcoef(dfFinal['averagerating'], dfFinal['profit']))

# fig,ax = plt.subplots(figsize = (10,10),
#                       ncols = 1, nrows = 1)
sns.jointplot(x=x, y=y,kind = 'reg', height = 15, color = 'red', joint_kws = {'line_kws' : {'color':'blue'}});
ax = plt.gca()
ax.set_title('Profit Vs Average IMDB Rating: Shaded by Production Budget');
# ax.legend([f"The Correlation is {np.corrcoef(dfFinal['averagerating'], dfFinal['profit'])[0][1]}"])
```


    array([[nan, nan],
           [nan,  1.]])



![png](output_53_1.png)


<h2>
    Movie Release Dates
</h2>
<p>
    Overall, it appears that there's isn't any industry wide benefit to releasing a movie on any given month or day of the week. However, it's clear that there's a strong preference for movies coming out on Friday. In general, Friday release dates don't have any material benefit but because it appears to be an industry standard, there is also a strong preference for Friday among the most profitable movies. I would also encorage Microsoft to release any movies on fridays. Following the industry standard is smart in this case because there may not be a way to determine if block busters make more money because they come out on Friday or if blockbusters are chosen to be released on Friday and they were going to make money anyways.
</p>


```python
#https://www.interviewqs.com/ddi_code_snippets/extract_month_year_pandas

thing1 = sns.jointplot(x = 'release_month', y ='profit', data = df2,kind = 'reg', height = 15,joint_kws = {'line_kws' : {'color':'red'}})
ax = plt.gca()
ax.set_title('Movie Profit By Release Month')
ax.set_xlabel('Month')
ax.set_ylabel('Profit')
thing2 = sns.jointplot(x = 'release_day', y ='profit', data = df2, height = 15,kind = 'resid' )
ax1 = plt.gca()
ax1.set_title('Movie Profit By Release Day')
ax1.set_xlabel('Day Of The Week')
ax1.set_ylabel('Profit')
thing3 = sns.jointplot(x = 'release_month', y ='profit', data = df2[:100], height =15, kind = 'reg',joint_kws = {'line_kws' : {'color':'red'}} )
ax2 = plt.gca()
ax2.set_title('Movie Profit By Release Month')
ax2.set_xlabel('Month')
ax2.set_ylabel('Profit')
thing4 = sns.jointplot(x = 'release_day', y ='profit', data = df2[:100], height = 15, kind = 'resid' )
ax3 = plt.gca()
ax3.set_title('Movie Profit By Release Day')
ax3.set_xlabel('Day Of The Week')
ax3.set_ylabel('Profit')
```




    Text(56.775000000000006, 0.5, 'Profit')




![png](output_55_1.png)



![png](output_55_2.png)



![png](output_55_3.png)



![png](output_55_4.png)



```python

```


```python

```
