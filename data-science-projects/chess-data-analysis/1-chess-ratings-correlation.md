# First article about api calls and storage inside MySQL database

--------------------------------------

In order to conduct any analysis, we'll need a dataset. My approach will be to build it through the API (Automated Program Interface) of chess.com. Lichess.org could be a nice as well, being the second biggest chess platform with an opensource mindset, and being the website giving the most information readily through their API. But I chose chess.com because it is the biggest online chess website, and most titled players do have an account at chess.com whereas I'm not sure how many of them do have an active account on Lichess.

There are a few questions we are looking an answer to:
- Do online and official ratings correspond well to each other?
- Should chess openings be evaluated the same way at higher and lower levels?
- Can we automatically identify a player's playing style? And are there potential matches between playing styles and opening choices?

We will tackle these questions in order. First, we will need data about online and official ratings of chess players. Since it is difficult to find information about official ratings of chess usernames, let alone reliable information, we will focus on titled players. In chess, titled players are players who completed certain achievements in official FIDE tournaments. The higher the title, the more difficult the achievements. This means information about the rating of these players should be the most reliable we can get, and so we'll work with them. All this will be done via requests to chess.com's API.

## Grabbing our initial dataset


```Markdown
gather list of titled players
print number of players per title
```

A number of usernames were gathered, and even the smallest categories seem nicely filled, a good start. We'll now prepare few functions to format the information about them into a valid entry and then store it. Since we will likely need to look at this data from different angles in later stages of this project, storing that information in a relational database seems like a good idea. I've therefore set up a MySQL database aside, and I'll use python to both push data in and retrieve data out of it.
```Markdown
def function_to_prepare_entries():
  return 'oh yeah'
  
def

def
```

Now the functions are ready, we can simply loop over our players, make the right calls to chess.com's API, and store that data.
```Markdown
#loop over players and gather data about them

#plot something to check that data has been retrieved
```

```Markdown
#store that data in MySQL
#print something to show that the database is nicely filled
```

## Cleaning the dataset


## Analyzing the dataset


## Conclusion
