First, we need to build the datasets we'll be using for the whole project. The data we need and why have aready been discussed in the project presentation article [link].

About the more technical details, we saw see that in every step we will be able to work from simple tabular data. However, the data will have to be presented from different angles. In order to accomodate for all this, we'll build our database with MySQL. All MySQL operations will be made from python thanks to the mysql.connector module.

For our first problem, we simply need many players with access to both their online ratings and an official reliable rating. For non-titled players it's not easy to gather FIDE (international chess federation) ratings, and also it's difficult to know whether that rating is accurate or if the players are still getting better, so we'll dismiss that data as inaccurate.

Through chess.com's api, we can access a list of titled players registered. We'll use stats of their accounts to find if they are active or not, and we'll also store the index of their archived games to save time for the next part.


```python
import requests
import json
import mysql.connector
from datetime import datetime

titles = ['GM', 'WGM', 'IM', 'WIM', 'FM', 'WFM', 'CM', 'WCM']
titled_list = dict()
for t in titles:
    api_url = 'https://api.chess.com/pub/titled/{}'.format(t)
    data = requests.get(api_url).json()
    titled_list[t] = {p: {} for p in data['players']}
```

We can have a quick check of the distribution of the accounts we gathered, to get an rough idea of what we'll work with:


```python
sum_players = 0
for t in titles:
    n = len(titled_list[t].keys())
    print(t, ' => ', n, 'players registered')
    sum_players += n
print('\nTotal number of players registered => ', sum_players)
```

    GM  =>  1281 players registered
    WGM  =>  182 players registered
    IM  =>  1711 players registered
    WIM  =>  313 players registered
    FM  =>  2556 players registered
    WFM  =>  448 players registered
    CM  =>  775 players registered
    WCM  =>  224 players registered
    
    Total number of players registered =>  7490


Looks nice so far!
Time to actually gather the information. We'll loop through all players, ignore them if they are not active, and gather information about their elos and get the index of their game archives.

First we'll write two functions to ease up the bulk of our code. One to make an api call and prepare the data to be inserted in our database...


```python
#Function to prepare an entry to be inserted in mysql
def player_entry(name, title):
    entry = [name, title]
    p_stats = requests.get('https://api.chess.com/pub/player/{}/stats'.format(p)).json()
    try:
        entry.append(p_stats['fide'])
    except:
        entry.append(None)
    try:
        entry.append(p_stats['chess_rapid']['last']['rating'])
    except:
        entry.append(None)
    try:
        entry.append(p_stats['chess_rapid']['best']['rating'])
    except:
        entry.append(None)
    try:
        entry.append( sum([p_stats['chess_rapid']['record'][n] for n in ['win', 'loss', 'draw']]) )
    except:
        entry.append(None)
    try:
        entry.append(p_stats['chess_blitz']['last']['rating'])
    except:
        entry.append(None)
    try:
        entry.append(p_stats['chess_blitz']['best']['rating'])
    except:
        entry.append(None)
    try:
        entry.append( sum([p_stats['chess_blitz']['record'][n] for n in ['win', 'loss', 'draw']]) )
    except:
        entry.append(None)
    try:
        entry.append(int(p_stats['chess_bullet']['last']['rating']))
    except:
        entry.append(None)
    try:
        entry.append(int(p_stats['chess_bullet']['best']['rating']))
    except:
        entry.append(None)
    try:
        entry.append( sum([p_stats['chess_bullet']['record'][n] for n in ['win', 'loss', 'draw']]) )
    except:
        entry.append(None)
   #While we're there, we should also get his game archive links
    try:
        game_history = requests.get('https://api.chess.com/pub/player/{}/games/archives'.format(p)).json()
    except:
        game_history = None
    #Extracting only the month format as 'YYYYMM'
    archives = [date[-8:-3] + date[-2:] for date in game_history['archives']]
    archives.sort(reverse = True)
    entry.append(datetime.today().strftime('%Y%m'))
    entry.append(archives)
    return entry
```

... And the second one to insert that entry in our database.


```python
def sql_connect():
    #Create connection to the database
    try:
        conn = mysql.connector.connect(host='localhost',
                                      database = 'chess_project',
                                      user = 'dfuu',
                                      password = 'p@s8my_SQL')
        cursor = conn.cursor()
    except:
        print("Couldn't connect to database")
        raise
    return conn, cursor

def sql_dc(co, cu):
    cu.close()
    co.close()
    return

def data_push_to_sql(entry, conn, cursor):
    #part of the function where we deal with the mysql data manipulation
    query = 'INSERT INTO Players(player_name,title,fide,rapid_elo_last,rapid_elo_best,rapid_n_games,\
                                blitz_elo_last,blitz_elo_best,blitz_n_games,\
                                bullet_elo_last,bullet_elo_best,bullet_n_games,last_updated,archives)'\
                                'VALUES(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)'
    #entry should be a tuple
    entry[-1] = str(entry[-1])
    entry = tuple(entry)
    cursor.execute(query, entry)
    #Validate change in database and close everything
    conn.commit()
    return
```

The external part of creating the MySQL database is left out of here: It is as straightforward as installing and creating a MySQL database, and making sure the MySQL server is up and running before executing this script.

We can now simply download the information of the players to build the first part of our dataset.


```python
n_deleted, n_checked = 0, 0
for t in titles:
    for p in titled_list[t]:
        # Deleting players with a closed account
        try:
            p_data = requests.get('https://api.chess.com/pub/player/{}'.format(p)).json()
            if p_data['status'] in ['closed', 'closed:fair_play_violations']:
                try:
                    del titled_list[t][p]
                    n_deleted += 1
                except:
                    print("Couldnt delete", p,'from the dictionary')
            else:
                #pushing that data in our database
                co, cu = sql_connect()
                data_push_to_sql(player_entry(p, t), co, cu)
                sql_dc(co, cu)
        except:
            pass
        #Printing completion status information
        n_checked += 1
        print('Checked: {}/{}'.format(n_checked, sum_players), ' / Deleted players:', n_deleted, end='\r')
```

    Checked:7490/7490  / Deleted players: 0


```python
import mysql.connector
import pandas as pd

conn, cursor = sql_connect()
df = pd.read_sql('SELECT * FROM Players', con = conn)
sql_dc(conn, cursor)

df.head()
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
      <th>player_id</th>
      <th>player_name</th>
      <th>title</th>
      <th>fide</th>
      <th>rapid_elo_last</th>
      <th>rapid_elo_best</th>
      <th>rapid_n_games</th>
      <th>blitz_elo_last</th>
      <th>blitz_elo_best</th>
      <th>blitz_n_games</th>
      <th>bullet_elo_last</th>
      <th>bullet_elo_best</th>
      <th>bullet_n_games</th>
      <th>last_updated</th>
      <th>archives</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1252</td>
      <td>124chess</td>
      <td>GM</td>
      <td>NaN</td>
      <td>2294.0</td>
      <td>2350.0</td>
      <td>7.0</td>
      <td>2662.0</td>
      <td>2830.0</td>
      <td>2308.0</td>
      <td>2419.0</td>
      <td>2605.0</td>
      <td>142.0</td>
      <td>202008</td>
      <td>['/202008', '/202007', '/202006', '/202005', '...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1253</td>
      <td>1977ivan</td>
      <td>GM</td>
      <td>2641.0</td>
      <td>2423.0</td>
      <td>2556.0</td>
      <td>6.0</td>
      <td>2683.0</td>
      <td>2798.0</td>
      <td>456.0</td>
      <td>2500.0</td>
      <td>2075.0</td>
      <td>2.0</td>
      <td>202008</td>
      <td>['/202008', '/202007', '/202006', '/202005', '...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1254</td>
      <td>1stsecond</td>
      <td>GM</td>
      <td>2582.0</td>
      <td>2413.0</td>
      <td>2608.0</td>
      <td>14.0</td>
      <td>2789.0</td>
      <td>2932.0</td>
      <td>10035.0</td>
      <td>2654.0</td>
      <td>2872.0</td>
      <td>1901.0</td>
      <td>202008</td>
      <td>['/202008', '/202007', '/202006', '/202005', '...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1255</td>
      <td>2nd_life</td>
      <td>GM</td>
      <td>2450.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2508.0</td>
      <td>2593.0</td>
      <td>4112.0</td>
      <td>2304.0</td>
      <td>2382.0</td>
      <td>489.0</td>
      <td>202008</td>
      <td>['/202008', '/202007', '/202006', '/202005', '...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1256</td>
      <td>2vladimirovich90</td>
      <td>GM</td>
      <td>2727.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2984.0</td>
      <td>3084.0</td>
      <td>1722.0</td>
      <td>3155.0</td>
      <td>3157.0</td>
      <td>601.0</td>
      <td>202008</td>
      <td>['/202008', '/202007', '/202006', '/202005', '...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.describe()
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
      <th>player_id</th>
      <th>fide</th>
      <th>rapid_elo_last</th>
      <th>rapid_elo_best</th>
      <th>rapid_n_games</th>
      <th>blitz_elo_last</th>
      <th>blitz_elo_best</th>
      <th>blitz_n_games</th>
      <th>bullet_elo_last</th>
      <th>bullet_elo_best</th>
      <th>bullet_n_games</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>15569.000000</td>
      <td>8927.000000</td>
      <td>9013.000000</td>
      <td>8273.000000</td>
      <td>9013.000000</td>
      <td>14611.00000</td>
      <td>14521.000000</td>
      <td>14611.000000</td>
      <td>12054.000000</td>
      <td>11628.000000</td>
      <td>12054.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>9036.000000</td>
      <td>1642.378515</td>
      <td>2066.778320</td>
      <td>2178.163302</td>
      <td>48.842117</td>
      <td>2367.40271</td>
      <td>2468.536809</td>
      <td>2280.048046</td>
      <td>2223.318566</td>
      <td>2337.937221</td>
      <td>1457.044052</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4494.527506</td>
      <td>1092.500734</td>
      <td>335.725245</td>
      <td>332.428130</td>
      <td>334.907275</td>
      <td>306.46181</td>
      <td>308.642739</td>
      <td>4670.457208</td>
      <td>341.448945</td>
      <td>350.524602</td>
      <td>4528.794197</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1252.000000</td>
      <td>0.000000</td>
      <td>400.000000</td>
      <td>819.000000</td>
      <td>1.000000</td>
      <td>415.00000</td>
      <td>613.000000</td>
      <td>1.000000</td>
      <td>513.000000</td>
      <td>673.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5144.000000</td>
      <td>0.000000</td>
      <td>1863.000000</td>
      <td>2000.000000</td>
      <td>4.000000</td>
      <td>2200.00000</td>
      <td>2302.000000</td>
      <td>178.000000</td>
      <td>2029.000000</td>
      <td>2137.000000</td>
      <td>36.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>9036.000000</td>
      <td>2281.000000</td>
      <td>2090.000000</td>
      <td>2200.000000</td>
      <td>10.000000</td>
      <td>2394.00000</td>
      <td>2494.000000</td>
      <td>742.000000</td>
      <td>2259.000000</td>
      <td>2376.000000</td>
      <td>212.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>12928.000000</td>
      <td>2417.000000</td>
      <td>2300.000000</td>
      <td>2409.000000</td>
      <td>25.000000</td>
      <td>2571.00000</td>
      <td>2677.000000</td>
      <td>2422.500000</td>
      <td>2450.000000</td>
      <td>2566.000000</td>
      <td>1015.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>16820.000000</td>
      <td>3411.000000</td>
      <td>2960.000000</td>
      <td>3045.000000</td>
      <td>16493.000000</td>
      <td>3234.00000</td>
      <td>3332.000000</td>
      <td>79270.000000</td>
      <td>3231.000000</td>
      <td>3357.000000</td>
      <td>97547.000000</td>
    </tr>
  </tbody>
</table>
</div>



And here we are. The data is nicely imported in our MySQL database, and we just loaded it from there. We have all the columns we wanted, the information seems correctly stored, however we can see that we have some NaN for elos right from the start.

A quick look at the describe() output shows that we have plenty of missing values for fide and rapid games, and quite a bit for blitz and bullet. We also have some false values when we can see there is at least one 0 in fide, which is impossible, and some rapid/blitz/bullet elos are ridiculously low. Unsurprisingly, it requires cleaning.
