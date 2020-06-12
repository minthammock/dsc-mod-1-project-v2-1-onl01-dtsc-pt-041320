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
import matplotlib.ticker as tick
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

<h2>How much should you spend on making a movie</h2>


```python
def reformat_large_tick_values(tick_val, pos):
    """
    Turns large tick values (in the billions, millions and thousands) such as 4500 into 4.5K and also appropriately turns 4000 into 4K (no zero after the decimal).
    """
    if tick_val >= 1000000000:
        val = round(tick_val/1000000000, 1)
        new_tick_format = '{:}B'.format(val)
    elif tick_val >= 1000000:
        val = round(tick_val/1000000, 1)
        new_tick_format = '{:}M'.format(val)
    elif tick_val >= 1000:
        val = round(tick_val/1000, 1)
        new_tick_format = '{:}K'.format(val)
    elif tick_val < 1000:
        new_tick_format = round(tick_val, 1)
    else:
        new_tick_format = tick_val

    # make new_tick_format into a string value
    new_tick_format = str(new_tick_format)
    
    # code below will keep 4.5M as is but change values such as 4.0M to 4M since that zero after the decimal isn't needed
    index_of_decimal = new_tick_format.find(".")
    
    if index_of_decimal != -1:
        value_after_decimal = new_tick_format[index_of_decimal+1]
        if value_after_decimal == "0":
            # remove the 0 after the decimal point since it's not needed
            new_tick_format = new_tick_format[0:index_of_decimal] + new_tick_format[index_of_decimal+2:]
            
    return new_tick_format
```


```python

```


```python
display(np.corrcoef(dfFinal['production_budget'], dfFinal['profit'])[0][1])

sns.jointplot(x = 'production_budget', y ='profit', data = dfFinal, height = 10, kind = 'reg' ,joint_kws = {'line_kws' : {'color':'red'}});
ax = plt.gca();
ax.set_title("Profit Vs Production Budget");
ax.set_xlabel('Production Budget [$]')

ax.set_ylabel('Profit [$]')
ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
ax.xaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
plt.tight_layout()

```


    0.6060881813278919



![png](output_29_1.png)


<p>
    Notice the positive correlation of <strong>0.606</strong>. This is highly significant the strongest positive relationship found with Protit out of all the numerical columns. But what does this mean in terms scale?
</p>
<br>
In the next cell we calculate the quartiles, or percentage thresholds to see for what production budgets yield the greatest profits industry wide.
<br>


```python
def percentProfit(array):
    quartileValues = dfFinal['production_budget'].quantile(array)
    profits = [dfFinal.loc[dfFinal['production_budget'] > x]['profit'].sum()/dfFinal['profit'].sum() for x in quartileValues]
    profitsDiff = [profits[i-1] - profits[i] for i in range(len(profits))]
    df = pd.DataFrame([])
    df['production'] = quartileValues
    df['profits'] = profits
    df['profits'][0] = 1
    display(df)
    return df

test = percentProfit([0,.05,.1,.15, .2,.25, .3,.35, .4,.45, .5,.55, .6,.65, .7,.75, .8,.85, .9,.95, 1])
fig = plt.figure(figsize=(15,10))
ax = sns.lineplot(data = test, x = 'production', y = 'profits',marker = 'x', markersize = 10, markerfacecolor = 'red', 
                  markeredgecolor = 'red', markeredgewidth = 3)
ax.set_title("Cumulative Percent Profit Vs Production Budget (5% quartiles)");
ax.set_xlabel('Production Budget [$]')
ax.set_ylabel('Cumulative Percent Total Profit')
ax.xaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));

# type(dfFinal['production_budget'].quantile([.8,.9]))
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
      <th>production</th>
      <th>profits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0.00</th>
      <td>1100.0</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>0.05</th>
      <td>500000.0</td>
      <td>0.994683</td>
    </tr>
    <tr>
      <th>0.10</th>
      <td>1000000.0</td>
      <td>0.990041</td>
    </tr>
    <tr>
      <th>0.15</th>
      <td>2000000.0</td>
      <td>0.985248</td>
    </tr>
    <tr>
      <th>0.20</th>
      <td>3200000.0</td>
      <td>0.974985</td>
    </tr>
    <tr>
      <th>0.25</th>
      <td>5000000.0</td>
      <td>0.948195</td>
    </tr>
    <tr>
      <th>0.30</th>
      <td>6500000.0</td>
      <td>0.940437</td>
    </tr>
    <tr>
      <th>0.35</th>
      <td>9000000.0</td>
      <td>0.924737</td>
    </tr>
    <tr>
      <th>0.40</th>
      <td>11000000.0</td>
      <td>0.899568</td>
    </tr>
    <tr>
      <th>0.45</th>
      <td>14000000.0</td>
      <td>0.874119</td>
    </tr>
    <tr>
      <th>0.50</th>
      <td>16000000.0</td>
      <td>0.850402</td>
    </tr>
    <tr>
      <th>0.55</th>
      <td>20000000.0</td>
      <td>0.801023</td>
    </tr>
    <tr>
      <th>0.60</th>
      <td>24000000.0</td>
      <td>0.784709</td>
    </tr>
    <tr>
      <th>0.65</th>
      <td>27500000.0</td>
      <td>0.758118</td>
    </tr>
    <tr>
      <th>0.70</th>
      <td>32500000.0</td>
      <td>0.715624</td>
    </tr>
    <tr>
      <th>0.75</th>
      <td>40000000.0</td>
      <td>0.656578</td>
    </tr>
    <tr>
      <th>0.80</th>
      <td>50000000.0</td>
      <td>0.607609</td>
    </tr>
    <tr>
      <th>0.85</th>
      <td>60000000.0</td>
      <td>0.555364</td>
    </tr>
    <tr>
      <th>0.90</th>
      <td>79440000.0</td>
      <td>0.473875</td>
    </tr>
    <tr>
      <th>0.95</th>
      <td>120000000.0</td>
      <td>0.334809</td>
    </tr>
    <tr>
      <th>1.00</th>
      <td>425000000.0</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>



![png](output_31_1.png)


<h2>
    Answer for Question 1:
</h2>
In general production budget is strongly related to a movie's financial performance as shown by the correlation. Now when looking at how this trend breaks down on spending lines we see that the top 5% of high budget movies make more than 30% of the industry's money. Becuase of this <strong>I recomend allocating at least 120 million for production budget to be in the top 5%</strong>




<h1>
    Question 2
</h1>
<br>
<h2>What are the most profitable Genres?</h2>
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
dfFinal.head(1)

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
      <th>213</th>
      <td>95</td>
      <td>2020-12-31</td>
      <td>Moonfall</td>
      <td>150000000</td>
      <td>0</td>
      <td>0</td>
      <td>-150000000.0</td>
      <td>2020</td>
      <td>12</td>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[['nan']]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
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
    ax.set_ylabel('Profit [$]')
    ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));

    ax.set_xticklabels(x, rotation= 60, fontdict={'horizontalalignment':'right'});
    plotnumber = plotnumber + 1
createAThing(dfFinal)
```


![png](output_38_0.png)


<h2>
    Answer for Question 2:
</h2>
<br>
<p>
    There appear to be several genres that are more commercialy viable, namely: <strong>Action, Adventure, Animation, family, fantasy, musical and Sci-fi genres</strong>.
<br>
<h3>
    In Depth Analysis for Question 2
</h3>

<br>
The following graph breaks down the correlations between production budget and profit but on a genre by genre basis.


```python
# Defeine a function that create all out plots based on genre


def createAThing(df):
    fig = plt.figure(figsize=(50,50))
    plotnumber = 1
    for i in genresDict.keys():
        x = df.loc[df['genresCode'] % genresDict[i] == 0]['production_budget'];
        y = df.loc[df['genresCode'] % genresDict[i] == 0]['profit'];
        z = df.loc[df['genresCode'] % genresDict[i] == 0]['averagerating']
        
        ax = sns.scatterplot(x, y, ax = plt.subplot(6,4, plotnumber), hue = z, palette = 'rocket_r');
        
        ax.set_title(f'{i.title()} Movies Profit Vs Rating',fontdict = {'size' : 28})
        ax.set_xlabel('Production Budget [$]',fontdict = {'size' : 28})
        ax.set_ylabel('Profit [$]', fontdict = {'size' : 28})
        ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
        ax.xaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));

#         ax.legend(np.corrcoef(x,y)[1])
        plt.tight_layout()

        
        plotnumber = plotnumber +1
createAThing(dfFinal)
```


![png](output_41_0.png)


<br>
<h1>
    Question 3:
</h1>
<br>
<h2>
    Who are the most profitable directors and writers overall? Per genre?
</h2>
<br>
<br>


```python
#get the names of the directors and writers
dfimdbName.loc[301124,'primary_name'] = 'James Cameron'
dfimdbName.loc[452518,'primary_name'] = 'James Cameron'
dfimdbName['isTopDirector'] = dfimdbName['nconst'].isin(dfFinal['directors'])
temp = dfimdbName.loc[dfimdbName['isTopDirector']]
dfFinal2 = dfFinal.merge(temp, how='left', left_on = 'directors', right_on = 'nconst')
dfFinal2.head(1)

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
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
      <th>isTopDirector</th>
      <th>isTopWriter</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>95</td>
      <td>2020-12-31</td>
      <td>Moonfall</td>
      <td>150000000</td>
      <td>0</td>
      <td>0</td>
      <td>-150000000.0</td>
      <td>2020</td>
      <td>12</td>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[['nan']]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (15,10))
    topDirectors = df.groupby('primary_name').mean().sort_values('profit', ascending = False).index[:25]
    directorRows = df['primary_name'].isin(topDirectors)
#     y = df.groupby('primary_name').mean().sort_values('profit', ascending = False)['profit'][:50]
    ax = sns.barplot(data = dfFinal2[directorRows], x = 'primary_name', y = 'profit',
                     ax = plt.subplot(1,1,plotnumber), palette='Blues_d', ci = 68, order = topDirectors)
    ax.set_title('Top 25 Directors Per Average Movie Profit')
    ax.set_xlabel('Director Name')
    ax.set_ylabel('Profit [$]')
    ax.set_xticklabels(ax.get_xticklabels(), rotation= 60, fontdict={'horizontalalignment':'right'});
    ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));

    plotnumber = plotnumber +1

createANewThing(dfFinal2)
```


![png](output_44_0.png)



```python
# Get the top writers
dfimdbName['isTopWriter'] = dfimdbName['nconst'].isin(dfFinal['writers'])
temp = dfimdbName.loc[dfimdbName['isTopWriter']]
dfFinal3 = dfFinal.merge(temp, how='left', left_on = 'writers', right_on = 'nconst')
dfFinal3.head(1)

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
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
      <th>isTopDirector</th>
      <th>isTopWriter</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>95</td>
      <td>2020-12-31</td>
      <td>Moonfall</td>
      <td>150000000</td>
      <td>0</td>
      <td>0</td>
      <td>-150000000.0</td>
      <td>2020</td>
      <td>12</td>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[['nan']]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (15,10))
    topWriters = df.groupby('primary_name').mean().sort_values('profit', ascending = False).index[:25]
    writerRows = df['primary_name'].isin(topWriters)
#     y = df.groupby('primary_name').mean().sort_values('profit', ascending = False)['profit'][:50]
    ax = sns.barplot(data = dfFinal2[writerRows], x = 'primary_name', y = 'profit',
                     ax = plt.subplot(1,1,plotnumber), palette='Reds_d', ci = 68, order = topWriters)
    ax.set_title('Top 25 Writers Per Average Movie Profit')
    ax.set_xlabel('Writer Name')
    ax.set_ylabel('Profit [$]')
    ax.set_xticklabels(ax.get_xticklabels(), rotation= 60, fontdict={'horizontalalignment':'right'});
    ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));

    plotnumber = plotnumber +1

createANewThing(dfFinal2)
```


![png](output_46_0.png)


<br>
<h2>
    Answer for Question 3:
</h2>
<p>
    The above graphs give a short list of both writers and directors who have have had vast commercial success. By averaging the profits accross their movies, we cut down on fluke victories. Every name in the visuals should be considered, with special attention given to those with existing report.  
<br>
    Note that in both catagories, people without an error bar only have one movie in this genre category. 
</p>
<br>
<br>

<h3>
    In Depth Analysis for Question 3
</h3>


```python
#get the names of the directors and writers
dfimdbName.loc[301124,'primary_name'] = 'James Cameron'
dfimdbName.loc[452518,'primary_name'] = 'James Cameron'
dfimdbName['isTopDirector'] = dfimdbName['nconst'].isin(dfFinal['directors'])
temp = dfimdbName.loc[dfimdbName['isTopDirector']]
dfFinal2 = dfFinal.merge(temp, how='left', left_on = 'directors', right_on = 'nconst')
dfFinal2.head(1)
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
      <th>nconst</th>
      <th>primary_name</th>
      <th>birth_year</th>
      <th>death_year</th>
      <th>primary_profession</th>
      <th>known_for_titles</th>
      <th>isTopDirector</th>
      <th>isTopWriter</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>95</td>
      <td>2020-12-31</td>
      <td>Moonfall</td>
      <td>150000000</td>
      <td>0</td>
      <td>0</td>
      <td>-150000000.0</td>
      <td>2020</td>
      <td>12</td>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[['nan']]</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (80,80))
    for i in genresDict.keys():
        topDirectors = df.loc[df['genresCode'] % genresDict[i] == 0].groupby('primary_name').mean().sort_values('profit', ascending = False).index[:25]
        directorRows = df['primary_name'].isin(topDirectors)
        ax = sns.barplot(data = df[directorRows].sort_values('profit', ascending = False), x = 'primary_name', y = 'profit',
                         ax = plt.subplot(6,4,plotnumber), palette='Blues_d', ci = 68, order = topDirectors)
        ax.set_title(f'Top 25 {i.title()} Directors', fontdict = {'fontsize':36})
        ax.set_xlabel('Director Name')
        ax.set_ylabel('Profit [$]')
        ax.set_xticklabels(ax.get_xticklabels(), rotation= 60, fontdict={'horizontalalignment':'right'});
        ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));

        plotnumber = plotnumber +1
    plt.tight_layout()

createANewThing(dfFinal2)
```

    C:\ProgramData\Anaconda3\lib\site-packages\seaborn\categorical.py:411: UserWarning: Attempting to set identical left == right == -0.5 results in singular transformations; automatically expanding.
      ax.set_xlim(-.5, len(self.plot_data) - .5, auto=None)
    


![png](output_50_1.png)


<br>
<p>
    This is a nice working list of the successful directors in each genres. Note that all people without error bars have only had one massive commercial success. But, now that we have a short list, we have a manageable amount of follow up research. Lets move on to the writers.
</p>
<br>
<br>


```python
dfimdbName['isTopWriter'] = dfimdbName['nconst'].isin(dfFinal['writers'])
temp = dfimdbName.loc[dfimdbName['isTopWriter']]
dfFinal3 = dfFinal.merge(temp, how='left', left_on = 'writers', right_on = 'nconst')

def createANewThing(df):
    plotnumber = 1
    fig,ax = plt.subplots(figsize = (80,80))
    for i in genresDict.keys():
        topWriters = df.loc[df['genresCode'] % genresDict[i] == 0].groupby('primary_name').mean().sort_values('profit', ascending = False).index[:25]
        writerRows = df['primary_name'].isin(topWriters)
        ax = sns.barplot(data = df[writerRows].sort_values('profit', ascending = False), x = 'primary_name', y = 'profit',
                         ax = plt.subplot(6,4,plotnumber), palette='Reds_d', ci = 68, order = topWriters)
        ax.set_title(f'Top 25 {i.title()} Writers', fontdict = {'fontsize':36})
        ax.set_xlabel('Writer Name')
        ax.set_ylabel('Profit [$]')
        ax.set_xticklabels(ax.get_xticklabels(), rotation= 60, fontdict={'horizontalalignment':'right'});
        ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));

        plotnumber = plotnumber +1
    plt.tight_layout()

createANewThing(dfFinal3)
```

    C:\ProgramData\Anaconda3\lib\site-packages\seaborn\categorical.py:411: UserWarning: Attempting to set identical left == right == -0.5 results in singular transformations; automatically expanding.
      ax.set_xlim(-.5, len(self.plot_data) - .5, auto=None)
    


![png](output_52_1.png)


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
ax.set_ylabel('Profit [$]', fontdict = {'size' : 20});
ax.set_xticklabels(sorted(x.unique()), rotation =45, fontdict={'horizontalalignment':'center'});
ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
plt.tight_layout()

```


![png](output_56_0.png)


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


# fig,ax = plt.subplots(figsize = (10,10),
#                       ncols = 1, nrows = 1)
sns.jointplot(x=x, y=y,kind = 'reg', height = 10, color = 'red', joint_kws = {'line_kws' : {'color':'blue'}});
ax = plt.gca()
ax.set_title('Profit Vs Average IMDB Rating');
ax.set_xlabel('Average IMDB Rating')
ax.set_ylabel('Profit [$]')
ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
plt.tight_layout()
# ax.legend([f"The Correlation is {np.corrcoef(dfFinal['averagerating'], dfFinal['profit'])[0][1]}"])
```


![png](output_59_0.png)


<h2>
    Movie Release Dates
</h2>
<p>
    Overall, it appears that there's isn't any industry wide benefit to releasing a movie on any given month or day of the week. However, it's clear that there's a strong preference for movies coming out on Friday. In general, Friday release dates don't have any material benefit but because it appears to be an industry standard, there is also a strong preference for Friday among the most profitable movies. I would also encorage Microsoft to release any movies on fridays. Following the industry standard is smart in this case because there may not be a way to determine if block busters make more money because they come out on Friday or if blockbusters are chosen to be released on Friday and they were going to make money anyways.
</p>


```python
#https://www.interviewqs.com/ddi_code_snippets/extract_month_year_pandas

thing1 = sns.jointplot(x = 'release_month', y ='profit', data = df2,kind = 'reg', height =8,joint_kws = {'line_kws' : {'color':'red'}})
ax = plt.gca()
ax.set_title('Movie Profit By Release Month')
ax.set_xlabel('Month')
ax.set_ylabel('Profit [$]')        
ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
plt.tight_layout()

thing2 = sns.jointplot(x = 'release_day', y ='profit', data = df2, height = 8,kind = 'resid' )
ax1 = plt.gca()
ax1.set_title('Movie Profit By Release Day')
ax1.set_xlabel('Day Of The Week')
ax1.set_ylabel('Profit [$]')
ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
plt.tight_layout()

thing3 = sns.jointplot(x = 'release_month', y ='profit', data = df2[:100], height =8, kind = 'reg',joint_kws = {'line_kws' : {'color':'red'}} )
ax2 = plt.gca()
ax2.set_title('Movie Profit By Release Month')
ax2.set_xlabel('Month')
ax2.set_ylabel('Profit [$]')
ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
plt.tight_layout()

thing4 = sns.jointplot(x = 'release_day', y ='profit', data = df2[:100], height = 8, kind = 'resid' )
ax3 = plt.gca()
ax3.set_title('Movie Profit By Release Day')
ax3.set_xlabel('Day Of The Week')
ax3.set_ylabel('Profit [$]')
ax.yaxis.set_major_formatter(tick.FuncFormatter(reformat_large_tick_values));
plt.tight_layout()


```


![png](output_61_0.png)



![png](output_61_1.png)



![png](output_61_2.png)



![png](output_61_3.png)


<h1>
    Recomendations:
</h1><br><br>

<h2>
Invest more than 120 million and less than 300 million in a movie 
</h2>

* Movies with a budget of 120 million or more make up aroung 5% of all films, but account for 34% of the industries total revenue

<br><br>

<h2>
Make one of the following types of Movies:
</h2>

* Action
* Adventure
* Animation
* Family
* Fantasy
* Musical
* Sci-Fi
<br><br>

<h2>
Hire writers and directors from the above lists in the chosen genre 
</h2>




<h1>
    Refrences:
</h1>
<h2>
    There are lots of folks in the open source community who make the world go round. Here are the ones I used in this notebook. Go check them out!
</h2>

* https://www.interviewqs.com/ddi_code_snippets/extract_month_year_pandas
* http://jonathansoma.com/lede/data-studio/classes/small-multiples/long-explanation-of-using-plt-subplots-to-create-small-multiples/
* https://dfrieds.com/data-visualizations/how-format-large-tick-values.html
