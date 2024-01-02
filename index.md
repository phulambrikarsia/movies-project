# SQL Project: Analyzing Movie Data

### Author: Sia Phulambrikar

In this project, we dive into a database of Movies, unraveling insights through SQL. I start with broad exploratory analysis about the dataset, and then delve deeper into genre-specific characteristics of movies, characters and dialogues. Read on as we uncover fascinating insights about Hollywood and the Silver Screen :)  


Skills: SQL 

## 1. Importing Dependancies and Loading Data


```python
import pandas as pd
from sqlalchemy import create_engine
from sqlalchemy import inspect
```


```python
movie_engine = create_engine('sqlite:///movie_lines.db')
insp = inspect(movie_engine)
table_names = insp.get_table_names()
con = movie_engine.connect()
```

## 2. Exploratory Analysis: 
- How many movies are in the database?
- How many movies does each genre contain?
- What release years do the movies span?

![Screenshot 2023-12-22 at 11.22.57 PM.png](3cc638ef-7a7d-4c72-86fe-01bca3d231e0.png)

### Q1. How many movies are in the database?


```python
query = '''
SELECT COUNT(movie_id) as n_movies
FROM movies
'''
pd.read_sql_query(query, con)
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
      <th>n_movies</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>617</td>
    </tr>
  </tbody>
</table>
</div>



There are 617 movies in the database.

### Q2. How many movies does the database contain per genre?

Display results for the top 3 genres.


```python
query = '''
SELECT genres.name as genre_name, COUNT(movies.movie_id) as n_movies
FROM movies
JOIN movie_genre_linking ON movies.movie_id = movie_genre_linking.movie_id
JOIN genres ON movie_genre_linking.genre_id = genres.genre_id
GROUP BY genres.genre_id
ORDER BY n_movies DESC
LIMIT 3
'''
pd.read_sql_query(query, con)
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
      <th>genre_name</th>
      <th>n_movies</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>drama</td>
      <td>320</td>
    </tr>
    <tr>
      <th>1</th>
      <td>thriller</td>
      <td>265</td>
    </tr>
    <tr>
      <th>2</th>
      <td>comedy</td>
      <td>159</td>
    </tr>
  </tbody>
</table>
</div>



The database contains the most number of Drama movies, at 320, followed by Thriller at 265. The total number of genres in the database are 23 (same as the number of rows).

### Q3. What release years do the movies span?


```python
query = '''
SELECT MIN(year) as start_year, MAX(year) as end_year
FROM movies
'''
pd.read_sql_query(query, con)
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
      <th>start_year</th>
      <th>end_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1927</td>
      <td>2010</td>
    </tr>
  </tbody>
</table>
</div>



The dataset spans movies released between 1927-2010.

## 3. Analysis Questions: Digging Deeper
- Which are the highest rated movies?
- Which genre receives the best and worst average ratings?
- Which are the most popular movies from each genre?
- Are there movies with an all-male or all-female cast?

##### Understanding dialogues in movies
- What are the average number of conversations per movie? Does this vary by genre?
- Does the length of conversation vary by movie genre?

##### Delving into the Golden Age of Hollywood
- Which were the most popular movies defining the Golden Age of Hollywood?
- Which were the Top 5 characters during the Golden Age?

### Q1. What are the Top 5 Highest Rated Movies?


```python
query = '''
SELECT movie_id, title as movie_title, imdb_rating
FROM movies
ORDER BY imdb_rating DESC
LIMIT 5
'''
pd.read_sql_query(query, con)
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
      <th>movie_id</th>
      <th>movie_title</th>
      <th>imdb_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>457</td>
      <td>neuromancer</td>
      <td>9.3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>203</td>
      <td>the godfather</td>
      <td>9.2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>369</td>
      <td>the godfather: part ii</td>
      <td>9.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>463</td>
      <td>one flew over the cuckoo's nest</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>504</td>
      <td>schindler's list</td>
      <td>8.9</td>
    </tr>
  </tbody>
</table>
</div>



Movies with the highest ratings are Neuromancer, The Godfather, The Godfather Part II, One Flew Over the Cuckoo's Nest and Schindler's List. The highest rated movie in the database, Neuromancer, has an IMDB rating of 9.3.

### Q2. What genre recives the best and worst average ratings?


```python
query = '''
WITH MovieGenres (movie_id, title, year, imdb_rating, imdb_votes, url, genre_id, genre_name)
AS 
    (SELECT movies.*, genres.*
    FROM movies
        JOIN movie_genre_linking ON movies.movie_id = movie_genre_linking.movie_id
        JOIN genres ON movie_genre_linking.genre_id = genres.genre_id
    )
SELECT genre_id, genre_name, AVG(imdb_rating) as avg_rating
FROM MovieGenres
GROUP BY genre_id
ORDER BY avg_rating DESC
'''
pd.read_sql_query(query,con)
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
      <th>genre_id</th>
      <th>genre_name</th>
      <th>avg_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>18</td>
      <td>film-noir</td>
      <td>8.400000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11</td>
      <td>war</td>
      <td>7.756522</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>biography</td>
      <td>7.684000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>21</td>
      <td>history</td>
      <td>7.547619</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>sport</td>
      <td>7.325000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>23</td>
      <td>musical</td>
      <td>7.275000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>19</td>
      <td>drama</td>
      <td>7.236562</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>romance</td>
      <td>7.050758</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>music</td>
      <td>7.015385</td>
    </tr>
    <tr>
      <th>9</th>
      <td>6</td>
      <td>crime</td>
      <td>7.001379</td>
    </tr>
    <tr>
      <th>10</th>
      <td>16</td>
      <td>mystery</td>
      <td>6.968000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>8</td>
      <td>animation</td>
      <td>6.962500</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>family</td>
      <td>6.956250</td>
    </tr>
    <tr>
      <th>13</th>
      <td>24</td>
      <td>adventure</td>
      <td>6.851402</td>
    </tr>
    <tr>
      <th>14</th>
      <td>10</td>
      <td>comedy</td>
      <td>6.751572</td>
    </tr>
    <tr>
      <th>15</th>
      <td>3</td>
      <td>western</td>
      <td>6.741667</td>
    </tr>
    <tr>
      <th>16</th>
      <td>15</td>
      <td>thriller</td>
      <td>6.704906</td>
    </tr>
    <tr>
      <th>17</th>
      <td>12</td>
      <td>sci-fi</td>
      <td>6.665179</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2</td>
      <td>fantasy</td>
      <td>6.612162</td>
    </tr>
    <tr>
      <th>19</th>
      <td>22</td>
      <td>documentary</td>
      <td>6.600000</td>
    </tr>
    <tr>
      <th>20</th>
      <td>17</td>
      <td>short</td>
      <td>6.560000</td>
    </tr>
    <tr>
      <th>21</th>
      <td>20</td>
      <td>action</td>
      <td>6.544304</td>
    </tr>
    <tr>
      <th>22</th>
      <td>14</td>
      <td>adult</td>
      <td>6.300000</td>
    </tr>
    <tr>
      <th>23</th>
      <td>13</td>
      <td>horror</td>
      <td>6.057143</td>
    </tr>
  </tbody>
</table>
</div>



From the above table, we see that Film Noir appears to be the most popular genre on average, with an average IMDB rating of 8.4. War and Biography movies also are rated quite highly on average. Genres rated lowest, on average, are Horror, Adult and Action, each with average ratings below 6.6.

### Q3. Which are the best rated movies from the 5 most popular genres?

Here, we find the best rated movies focusing on the 5 most popular genres found above, which were Film Noir, War, Biography, History and Sport.


```python
query = '''
WITH MovieGenres(movie_id, title, year, imdb_rating, imdb_votes, url, genre_id, genre_name)
AS
(SELECT movies.*, genres.*
FROM movies
    JOIN movie_genre_linking as link
    ON movies.movie_id = link.movie_id
    
    JOIN genres ON genres.genre_id = link.genre_id
)

SELECT genre_id, genre_name, title as movie_title, MAX(imdb_rating) as imdb_rating
FROM MovieGenres
GROUP BY genre_id
ORDER BY AVG(imdb_rating) DESC
LIMIT 5
'''
pd.read_sql_query(query,con)
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
      <th>genre_id</th>
      <th>genre_name</th>
      <th>movie_title</th>
      <th>imdb_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>18</td>
      <td>film-noir</td>
      <td>sunset blvd.</td>
      <td>8.7</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11</td>
      <td>war</td>
      <td>schindler's list</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5</td>
      <td>biography</td>
      <td>schindler's list</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>21</td>
      <td>history</td>
      <td>schindler's list</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>sport</td>
      <td>raging bull</td>
      <td>8.4</td>
    </tr>
  </tbody>
</table>
</div>



On inspecting the highest rated movies from the most popular genres, we see some popular names pop up such as Schindler's List, the highest rated War movie. However, Schindler's List appears under multiple genres including Biography and History. This is because each movie is linked to multiple possible genres under the movie_genre_linking table. Thus, movies with high ratings like Neuromancer (9.3) and Godfather (9.2) appear mutliple times. 

### Q4. Are there movies with an all-male or all-female cast? If not, which movies have the highest male:female ratio?


```python
#movies with an all-male cast
query = '''
SELECT movie_id, title as movie_title, n_female, n_male

FROM 
    (SELECT characters.movie_id, movies.title, COUNT(CASE WHEN gender = 'F' THEN gender ELSE NULL END) as n_female, 
        COUNT(CASE WHEN gender = 'M' THEN gender ELSE NULL END) as n_male 
    FROM characters
    JOIN movies ON characters.movie_id = movies.movie_id
    GROUP by characters.movie_id
    )

WHERE n_female = 0 AND n_male > 0
ORDER BY n_male DESC
'''
pd.read_sql_query(query, con)
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
      <th>movie_id</th>
      <th>movie_title</th>
      <th>n_female</th>
      <th>n_male</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>247</td>
      <td>apocalypse now</td>
      <td>0</td>
      <td>8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>170</td>
      <td>reservoir dogs</td>
      <td>0</td>
      <td>7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>525</td>
      <td>south park: bigger longer &amp; uncut</td>
      <td>0</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>536</td>
      <td>dr. strangelove or: how i learned to stop worr...</td>
      <td>0</td>
      <td>7</td>
    </tr>
    <tr>
      <th>4</th>
      <td>37</td>
      <td>the boondock saints</td>
      <td>0</td>
      <td>6</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>68</th>
      <td>277</td>
      <td>la battaglia di algeri</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>69</th>
      <td>315</td>
      <td>dark city</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>70</th>
      <td>478</td>
      <td>predator</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>71</th>
      <td>488</td>
      <td>red white black &amp; blue</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>72</th>
      <td>572</td>
      <td>ticker</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>73 rows × 4 columns</p>
</div>



We can see that Apocalypse Now had the largest all-male cast in our dataset with an 8:0 male:female ratio, followed by Reservoir Dogs at 7:0. A simple google search about the Apocalypse Now cast would reveal that the results are correct - the film did comprise of an all-male cast. In total, there are 73 such movies with an all-male cast in the dataset.


```python
query = '''
SELECT movie_id, title AS movie_title, n_female, n_male

FROM 
    (SELECT characters.movie_id, movies.title, COUNT(CASE WHEN gender = 'F' THEN gender ELSE NULL END) as n_female, 
        COUNT(CASE WHEN gender = 'M' THEN gender ELSE NULL END) as n_male 
    FROM characters
    JOIN movies ON characters.movie_id = movies.movie_id
    GROUP by characters.movie_id
    )

WHERE n_male = 0 AND n_female > 0
ORDER BY n_female DESC
'''
pd.read_sql_query(query, con)
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
      <th>movie_id</th>
      <th>movie_title</th>
      <th>n_female</th>
      <th>n_male</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>51</td>
      <td>drop dead gorgeous</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>234</td>
      <td>agnes of god</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>384</td>
      <td>heavenly creatures</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>393</td>
      <td>hellraiser iii: hell on earth</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>138</td>
      <td>my mother dreams the satan's disciples in new ...</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>134</td>
      <td>metropolis</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>459</td>
      <td>the nightmare before christmas</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>500</td>
      <td>salt of the earth</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



There are only 7 movies with an all female dataset. The top of this list is Drop Dead Gorgeous, a 1999 film which indeed has an all female cast. This is tied with Agnes of God, Heavenly Creatures and Hellraiser III: Hell on Earth.

### Q5. What are the average number of conversations per movie? Which movies have the maximum number of conversations, and which have the minimum?


```python
#average number of conversations overall
query = '''
WITH conversation_count (movie_id, n_conversations) 
AS 
(
SELECT movie_id, COUNT(conversation_id) as n_conversations
FROM conversations
GROUP BY movie_id
)

SELECT AVG(n_conversations) as avg_conversations
FROM conversation_count;
'''
pd.read_sql_query(query, con)
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
      <th>avg_conversations</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>134.679092</td>
    </tr>
  </tbody>
</table>
</div>




```python
#movies with the highest number of conversations
query = '''
WITH conversation_count (movie_id, n_conversations, title) 
AS 
(
SELECT movies.movie_id, COUNT(conversation_id) as n_conversations, movies.title
FROM conversations
INNER JOIN movies ON conversations.movie_id = movies.movie_id
GROUP BY conversations.movie_id
)

SELECT *
FROM conversation_count
WHERE n_conversations = (SELECT MAX(n_conversations) FROM conversation_count) 

UNION
SELECT *
FROM conversation_count
WHERE n_conversations = (SELECT MIN(n_conversations) FROM conversation_count)
;
'''
pd.read_sql_query(query, con)
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
      <th>movie_id</th>
      <th>n_conversations</th>
      <th>title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>289</td>
      <td>338</td>
      <td>casino</td>
    </tr>
    <tr>
      <th>1</th>
      <td>406</td>
      <td>1</td>
      <td>the jazz singer</td>
    </tr>
    <tr>
      <th>2</th>
      <td>602</td>
      <td>1</td>
      <td>what women want</td>
    </tr>
  </tbody>
</table>
</div>



The average number of conversations per movie is 134. The movie with the highest number of conversations (338) is Casino. The movies with the lowest (1) are the Jazz Singer and What Women Want. The Jazz Singer, interestingly, was one of the first movies to have a synchronized vocal and musical track, a milestone in the sound revolution. The movie was all-silent except just two minutes of vocal dialogue.

### Q6. Does the length of conversations vary by movie genre?


```python
query = '''
WITH line_counter 
AS
(SELECT conversation_id, COUNT(line_sort) as n_lines, lines.movie_id, genres.genre_id, genres.name
FROM lines
    JOIN movie_genre_linking 
    ON lines.movie_id = movie_genre_linking.movie_id

    JOIN genres
    ON movie_genre_linking.genre_id = genres.genre_id

GROUP BY conversation_id
)

SELECT genre_id, name, AVG(n_lines) as avg_lines
FROM line_counter
GROUP BY genre_id
ORDER BY avg_lines DESC
'''
pd.read_sql_query(query, con)
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
      <th>genre_id</th>
      <th>name</th>
      <th>avg_lines</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4</td>
      <td>sport</td>
      <td>13.279215</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>family</td>
      <td>13.278655</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>crime</td>
      <td>13.102732</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8</td>
      <td>animation</td>
      <td>12.964912</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>fantasy</td>
      <td>12.472645</td>
    </tr>
    <tr>
      <th>5</th>
      <td>3</td>
      <td>western</td>
      <td>12.229846</td>
    </tr>
    <tr>
      <th>6</th>
      <td>5</td>
      <td>biography</td>
      <td>11.513866</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>romance</td>
      <td>10.608152</td>
    </tr>
    <tr>
      <th>8</th>
      <td>12</td>
      <td>sci-fi</td>
      <td>10.496909</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>music</td>
      <td>9.912390</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>war</td>
      <td>9.907377</td>
    </tr>
    <tr>
      <th>11</th>
      <td>13</td>
      <td>horror</td>
      <td>9.429785</td>
    </tr>
    <tr>
      <th>12</th>
      <td>15</td>
      <td>thriller</td>
      <td>8.814720</td>
    </tr>
    <tr>
      <th>13</th>
      <td>10</td>
      <td>comedy</td>
      <td>8.718647</td>
    </tr>
    <tr>
      <th>14</th>
      <td>18</td>
      <td>film-noir</td>
      <td>7.795222</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>mystery</td>
      <td>7.543767</td>
    </tr>
    <tr>
      <th>16</th>
      <td>17</td>
      <td>short</td>
      <td>5.480176</td>
    </tr>
    <tr>
      <th>17</th>
      <td>20</td>
      <td>action</td>
      <td>4.967136</td>
    </tr>
    <tr>
      <th>18</th>
      <td>19</td>
      <td>drama</td>
      <td>4.791919</td>
    </tr>
    <tr>
      <th>19</th>
      <td>22</td>
      <td>documentary</td>
      <td>3.094340</td>
    </tr>
  </tbody>
</table>
</div>



The longest conversations, on average, occur in Sport genre movies, with Family and Crime movies being close runners-up. The shortest conversations, on the other hand, occur in Documentary films, followed by Drama and Action.

This could also be attributed to the way the dataset has been created - monologues by characters count as a single line in a "conversation". Thus, in documentaries, where one documentary narrator might be speaking multiple lines at once in a monological fashion, this might be considered a single line for the dataset, leading to the least number of lines per conversation.

### Q7. Which were the most popular movies defining the Golden Age of Hollywood?

The Golden Age of Hollywood was characterized by sound in movies, dominance of the "Big 5" studios, star power, glitz and glamour. The period popularly began in 1927 with the release of Jazz Singer, the first film to experiment with sound and faltered around 1950.


```python
query = '''
SELECT title, year, imdb_rating
FROM movies
WHERE year >= 1927 AND year <= 1950
ORDER BY imdb_rating DESC
LIMIT 6
'''

pd.read_sql_query(query, con)
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
      <th>year</th>
      <th>imdb_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>casablanca</td>
      <td>1942</td>
      <td>8.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>it's a wonderful life</td>
      <td>1946</td>
      <td>8.7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>sunset blvd.</td>
      <td>1950</td>
      <td>8.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>citizen kane</td>
      <td>1941</td>
      <td>8.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>all about eve</td>
      <td>1950</td>
      <td>8.5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>the third man</td>
      <td>1949</td>
      <td>8.5</td>
    </tr>
  </tbody>
</table>
</div>



The most popular IMDB rated movies from this period were Casablanca, It's a Wonderful Life, Sunset Blvd, Citizen Kane (which many refer to as the greatest movie ever made) and All About Eve. It's curious that Jazz Singer doesn't appear in this list. We can also look up the specific record for The Jazz Singer and compare its ratings to the Top 5 mentioned above. 


```python
query = '''
SELECT title, year, imdb_rating
FROM movies
WHERE title LIKE '%jazz%singer%'
'''
pd.read_sql_query(query, con)
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
      <th>year</th>
      <th>imdb_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>the jazz singer</td>
      <td>1927</td>
      <td>6.8</td>
    </tr>
  </tbody>
</table>
</div>



Interestingly, IMDB rating for the Jazz Singer is 6.8, much lower than the top 5 movies from this era listed above.

### Q8. Who were the most prominent characters from the Top 5 Golden Age Movies?


```python
query = '''
SELECT title AS movie_title, imdb_rating, name as character_name, n_lines
FROM
    (WITH line_counter 
    AS
        (SELECT movies.movie_id, movies.title, movies.imdb_rating, characters.character_id, name, COUNT(line_id) as n_lines
        FROM movies
            JOIN characters ON movies.movie_id = characters.movie_id
            JOIN lines ON characters.character_id = lines.character_id
        WHERE year >= 1927 AND year <= 1950
        GROUP BY characters.character_id
        ORDER BY imdb_rating DESC
        )
    
    SELECT *, ROW_NUMBER() over (PARTITION BY movie_id ORDER BY n_lines DESC) AS line_rank
    FROM line_counter
    GROUP BY character_id
    ORDER BY imdb_rating DESC)
WHERE line_rank <=2
LIMIT 10
'''

pd.read_sql_query(query,con)
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
      <th>movie_title</th>
      <th>imdb_rating</th>
      <th>character_name</th>
      <th>n_lines</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>casablanca</td>
      <td>8.8</td>
      <td>RICK</td>
      <td>281</td>
    </tr>
    <tr>
      <th>1</th>
      <td>casablanca</td>
      <td>8.8</td>
      <td>RENAULT</td>
      <td>135</td>
    </tr>
    <tr>
      <th>2</th>
      <td>it's a wonderful life</td>
      <td>8.7</td>
      <td>GEORGE</td>
      <td>275</td>
    </tr>
    <tr>
      <th>3</th>
      <td>it's a wonderful life</td>
      <td>8.7</td>
      <td>MARY</td>
      <td>57</td>
    </tr>
    <tr>
      <th>4</th>
      <td>sunset blvd.</td>
      <td>8.7</td>
      <td>GILLIS</td>
      <td>278</td>
    </tr>
    <tr>
      <th>5</th>
      <td>sunset blvd.</td>
      <td>8.7</td>
      <td>NORMA</td>
      <td>158</td>
    </tr>
    <tr>
      <th>6</th>
      <td>citizen kane</td>
      <td>8.6</td>
      <td>KANE</td>
      <td>221</td>
    </tr>
    <tr>
      <th>7</th>
      <td>citizen kane</td>
      <td>8.6</td>
      <td>LELAND</td>
      <td>93</td>
    </tr>
    <tr>
      <th>8</th>
      <td>all about eve</td>
      <td>8.5</td>
      <td>MARGO</td>
      <td>314</td>
    </tr>
    <tr>
      <th>9</th>
      <td>all about eve</td>
      <td>8.5</td>
      <td>EVE</td>
      <td>232</td>
    </tr>
  </tbody>
</table>
</div>



The table above gives us the two most prominent characters from the most popular Golden Age movies. I determined the most prominent characters by ordering them by their number of dialogues in the movie. For the movie Casablanca, for instance, we get 'Rick' and 'Renault' as the two most prominent characters. While the dataset doesn't include last names of characters, movie buffs might know that Rick Blaine was Humphrey Bogart's famous character in Casablanca, while Renault refers to Captain Louis Renault from the movie.
