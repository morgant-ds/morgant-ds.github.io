# Chess article - noob overview

-------------------------------
This is an overview of the project containing the run-through and conclusions, for a more in-depth look of how the project was conducted, it was broken down in a few articles:

[Building our initial dataset and initial analysis](chess-data-analysis/1-chess-ratings-correlation.md)

[Clean-up and initial analysis](chess-data-analysis/part-2-Analysis-cleaning-initial-dataset.md)

[Finishing to build our dataset via graph exploration](chess-data-analysis/part-3-finishing-dataset-with-graph-sampling.md)

-------------------------------

## Do online and official ratings correlate well?


A question often asked on chess forums is if it is possible to predict how much they could officially be rated given their online rating. The common answer is "Not really, but from this website you should remove XXX ELO and from this one YYY". In this section I'll try to simply observe if this kind of answer can be backed up by data somehow.

In order to only get online chess players for whom we have a reliable official rating to compare to, I chose to work through chess.com's API and with titled players. It is difficult to get reliable official ratings for non-titled usernames, but if we can't get any reasonable correlation with titled players there is no need to gather the data from non-titled as their ratings are typically much more volatile since many of these players are typically in the population range of improving players, meaning unaccurate ratings.

Our initial dataset had many problems with its entries. I decided to remove the data about *Rapid* time controls, since more than half of our players didn't play a single one of these. We had quite a bit of corrupted data in the FIDE rating column which was simply replaced by NaN, effectively relabelling them as missing data. The missing fide ratings were kept as is, since we want to analyze the correlation between this feature and other things I felt it was important to not replace these missing values arbitrarily and potentially affect our results. We also had a noticeable proportion of inactive players, which were simply deleted as they will not be useful for answering either our current question or the next ones.

*Show first graph with average ratings and "standard deviations" depending on titles*

Interestingly, we notice that the average ratings per title are actually nicely ordered. We'll plot online ratings versus fide ratings next

*figure*

Unfortunately, it seems that our dots are all over the place. I tried a simple linear regression on this data, and although it shows correlation, the standard deviation of around 150 fide rating when using it as a prediction tool shows that it's clearly not good. Especially with the hypothesis that this data should be much less volatile for titled players than for non-titled players. This hypothesis is based off two reasons:

- The lower the rating of a player, the more likely it is that this player is still improving, thus increasing the volatility of his results

- The better a player, the more consistent his play will be. A more consistent quality of play will lead to more consistent results, and since consistency in results should translate directly in more consistent ratings, their ratings should also display less variance.

There is not much point in trying other fitting models to this data, the dispersion in ratings is just too high to get a good precision.
I am now fairly confident to say that one cannot predict a potential FIDE rating from his online ratings. Well, one can, but the range is just too large for that prediction to be any useful or meaningful.

## Are chess openings relatively as good in the hands of a beginner or a grandmaster?

*Link to Hikaru Nakamura's video series about chess opening tier lists + introduction comments*

### Completing the dataset: Graph Sampling.


### A data scientist's chess opening tier list!
