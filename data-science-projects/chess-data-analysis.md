# **Chess data analysis**

### [Home](https://morgant-ds.github.io) > [Projects](https://morgant-ds.github.io/data-science-projects) 
-------------------------------
This page presents the story of this data science project without delving deep in detail. Please keep reading for an overview of the methodology and results of this study, or head over to the links below for a more in-depth look.

[Do online and official ratings correlate well?](chess-data-analysis/chess-ratings-correlation.md)

-------------------------------

# **Do online and official ratings correlate well?**

A very recurrent question on online chess forums is the possibility to predict a player's potential FIDE (Fédération Internationale Des Echecs, the official international chess organization) rating from his online ratings. Indeed, online chess is very popular and the majority of players don't actually belong to official clubs, let alone play official tournaments. Yet they are curious about how their skill level might actually compare to players in the official circuit.  
The answers they get is generally a bit vague, along the lines of "We can't really compare. But if you remove XXX rating from this online rating maybe you get an idea." This kind of answer is both not satisfactory enough, because the player can't know for sure, but also may very well be way too precise. This is what I will explore in the first part of this project.

In order to only get online chess players for whom I have a reliable official rating to compare to, I chose to work through chess.com's API and with titled players. It is difficult to get reliable official ratings for non-titled usernames, but if I can't get any reasonable correlation with titled players there is no need to gather the data from non-titled as their ratings are typically much more volatile. This assumption is based on two main arguments:  

- The lower the rating of a player, the more likely it is that this player is still improving, thus increasing the volatility of his results.  

- The better a player, the more consistent his play will be. A more consistent quality of play will lead to more consistent results, and since consistency in results should translate directly in more consistent ratings, their ratings should also display less variance.  

Also, this assumption is well shared in the chess community. Still, this is just a hypothesis. A study could be made to assess its relative truth, but since official ratings for non-titled players and especially low-rated players are hard to find reliably alongside their usernames we'll just assume that it is true.

Our initial dataset - a list of titled players registered on chess.com - had many problems with its entries. I decided to remove the data about *Rapid* time controls, since less than 10% of the players played at least 100 games. I also had quite a bit of corrupted data in the FIDE rating column which was simply replaced by NaN, effectively relabeling them as missing data. The missing fide ratings were kept as is, since I want to analyze the correlation between this feature and other things I felt it was important not to replace these missing values arbitrarily and potentially affect our results. I also had inactive players, which were simply deleted as they will not be useful for answering either our current question or the next ones.

![Mean and standard deviation vizualization of chess ratings](chess-data-analysis/chess-ratings-correlation/output_15_1.png)

Interestingly, we notice that the average ratings are nicely ordered by title. Also, a Pearson correlation was run on this data and showed clear correlation (with coefficient around 0.60) between fide and online ratings.
  
  
But unfortunately, the dots are all over the place. I tried a simple linear regression on this data and found a standard deviation of around 150 fide rating when using it as a prediction tool which is way too high if we want to use such a model to make any predictions.  
There is not much point in trying other fitting models to this data: **the dispersion in ratings is just too high to get a good precision**, as seen in the following figure:  

![Regression plot of online blitz/bullet vs FIDE ratings](chess-data-analysis/chess-ratings-correlation/output_31_1.png)

I am now fairly confident to say that one cannot predict a potential FIDE rating from his online ratings only. Well, one can, but the range is just too large for that prediction to be useful. However, I may develop other metrics to describe player's skill level later in this project. Maybe they will prove more useful to answer this question than the data we have so far.  
  
Sorry fellow chess players: if you want to assess your skill level in "over the board" tournaments you'll have to play a few of them!
