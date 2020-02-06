---
layout: post
title: Pivoting expectations in data analysis
subtitle: A journey through different hypotheses in archery data.
---



The scope of this data exploration and analysis changed several times over a couple of weeks. I originally requested World Archery API access without any particular data in mind, but during the emails with Chris Wells, my contact at World Archery, we agreed that an interesting place to start would be historical data regarding the Olympics, and how the Olympics affects arrow averages in tournaments before, during, and after the Olympics themselves.

# The Olympics

I began by making an API call that would return all competitions within a given time frame, starting off in the year the Rio Summer Olympics too place. It was this point when I started to realize the sort of state the data was in.
```json
{
  "pageInfo": {
    "totalResults": 1339,
    "resultsPerPage": 10,
    "page": 0
  },
  "items": [...
```
However, after a bit of looking, I realized a lot of these competition entries had no data associated with them. No results turned in. Luckily there was an option to perform the API call in such a way that only competitions with results would return.

```json
{
  "pageInfo": {
    "totalResults": 35,
    "resultsPerPage": 10,
    "page": 0
  },
  "items": [...
 ```
 Over 1300 results eliminated with no useful data to us. Only 35 events remained out of 1339 total events in 2016. So now all we need to do is pull the scores from each, get an average, and toss it on a plot to see where everything lines up at.

... Except there's still a couple things we're forgetting that requires a little explanation of Olympic format to understand. Olympic Archery competitions, at least for Recurve, operate under the Freestyle Recurve format. This format takes place outside, at 70 meters, with a 122cm target face, with scores ranging from 1 to 10, with zero for a miss. The data we're specifically looking at for comparing the Olympics to other events is the average of all arrows shot during an elimination match.

In our data, we have competitions like the Dublin 2016 World Archery Field Championships. Unfortunately, we can't really compare field archery to Freestyle Recurve for a number of reasons. First and foremost, it uses an entirely different score system, from 0 to 6 rather than 0 to 10. Secondly, field archers are often required to make strange shots, such as shooting uphill at a 20 degree angle, or standing on the side of a hill and shooting down at a target, and all from a variety of ranges. It would not be fair to compare Field and Freestyle Recurve.

But it doesn't stop there, as we have the Marrakesh 2016 Indoor Archery World Cup Stage. As the name suggests, this specific format of archery is called Indoor Archery, and the difference is more than the lack of wind. Indoor Archers shoot at a target that is 18 meters away instead of 70 meters, as well as at a smaller, 40cm target with three faces, one for each arrow of the set. Archers are typically so accurate in this format that they would often damage arrows if they shot at the same target face. So, with the closer range, different target size, and overall accuracy improvements, it would not be fair to compare Indoor archery with Freestyle Recurve.

There are a few other formats which also dont fit in well with Freestyle recurve, like Para Archery (think like Paralympics), that also shouldn't be directly compared.

After getting rid of all of the unsuitable formats, as well as the competitions that did not have sufficient data (such as including results about the match, but not recording specific arrow scores), we have reduced our 35 competitions down to... six. Not exactly a large sample size. 

In addition to the 2016 data, I also collected data from three years before and three years after, to see if I could catch any trends. Some years had more results than 2016 (2015 with 10, 2018 with 14) and one other year (2014) had 6 usable competitions. 2013 had no usable data at all. At this point I graphed all of the data I collected, originally starting with a line plot, with each year being a different line, but my low sample size did me no favors in this case. I then decided to group all of the competitions by month and display them all together.

![OlympicsAverages](https://i.imgur.com/lPdr8uQ.png)
*A note: I would have used notched boxplots here to give a confidence value on the medians, but with the low sample size here, several of the notches extend beyond the box, and just look ugly in my opinion.*

Well, the olympics appears to be a slightly above average event as far as this is concerned, but more interesting is the sort of sine wave pattern that the data shows, with peaks in april and late september, and a valley in july and over the holidays. The original question regarding the Olympics was a bust, and this strange seasonal bias was clearly what I should be exploring.

# The Seasonal Bias 

After discovering this for the first time, I didn't know what to think about it, so I sent an email back to Chris at World Archery, and he was equally surprised by what I found. He explained the valley over the holidays can be easily explained by the fact that it's the holidays, and that the major peak we see in September is due to the World Cup Finals, where the best in the world are playing at their best. He also offered a second mode of getting more data and more consistent averages, Qualifier rounds.

Each tournament has qualifier rounds to determine tournament seeding. The tournaments generally operate on a 72 arrow or 144 arrow, depending on what year they happened in, and always in sets of 36 (six arrows per end, with six ends). Strangely enough, qualifiers were much more consistently recorded than actual match scores. Not only was I able to get more scores per eligible match, I was also able to expand the years I could search from 2014-present, to 2000-present. This resulted in a large bump in data points from only a handful, to nearly 9000 individual competitors.

I was fairly happy with the function I created to gather this data. Here it is on [my github](https://github.com/VegaSera/World-Archery-Analysis/blob/master/WA_Arrow_Averages_Qualifiers.ipynb), but I have it below here in pseudocode format:
In order to get this data, the API call required a competition ID, an athlete ID, and a category code.

```python
for year in range(min_year, max_year):
  #Make the year's competition file, complete with competition IDs
  for competition_id in competitions['ID']:
    #From each competition we get the date.
    #Get the match data using our competition ID
    for competitor in match:
      #Get our unique competitor IDs, and return them to a list, filtering out the duplicates
    for athlete in athlete_list:
      #Get our athlete's qualification data for this competition
      #Append all of the useful data to a dataframe we set up outside the for loop.
```

So with a single function call, we could get every qualification arrow, from every competitor, from every match, from every competition, from every year in our year range, and with some data validation, of course, put it all into a single dataframe we can work with.

However, I underestimated exactly what I was doing, and the individual API calls for individual qualifier matches turned out to be a lot slower than I expected most of the time. My original dataframe for Recurve Men took 83 minutes to complete, with the vast majority of returned qualifier matches being empty or otherwise unusable. Unfortunately we cannot simply check a box that says "Has results" like we could with competition data.

Despite all that, we have our data with our much greater number of datapoints, and it's time to get to plotting it!

![Qualifiers](https://i.imgur.com/9f336pR.png)

# Pivotting again

Wait, what? Where's the cool sine wave? Where's the odd anomaly that we saw in the above graph? Sure, something is *certainly* happening in July, and the confidence values shown in our notches tell us it's quite unlikely that this is due to random chance. And it even still looks like a seasonal bias in the women's recurve graph, just... much less pronounced than it was in the original. But that dip in July is still definitely there.

Let's see if we can't figure out exactly what's going on here. Let's just make a few more graphs to explore and... oh...

![Non-World Level Qualifiers](https://i.imgur.com/bUPOsNu.png)

It appears I have greatly misinterpreted the data. The valley in July wasn't a seasonal thing, as least not something that transcends all levels of competitive archery. It's something that exists specifically **in the world cup level**.

However, it's at this point I reach a standstill with what I can find. I take all of the events that that correspond to this format and take place in july, run them through a function to get their qualifier arrow averages, drop the null values (for missing qualifier data), and after all that, I have very few options remaining. Medellin, Columbia in 2013 (Avg 8.52), Berlin 2018 (Avg 8.32) and Berlin 2019 (Avg 8.20), all of which averaging lower than what we expect. 

The only reason our july box isn't even lower is the inclusion of World Championship events with the other World-level competition levels. Each of the World Championship competitions comes in much closer to our expected median, New York City 2003 (Avg 9.04) and Copenhagen 2015 (Avg 8.74).

I tried all manner of explanation for this. At first I thought it might be weather related, given that it's july. In Medellin, Columbia during the dates of its tournament, the temperature did not rise above 80F/26.6C, and the wind was mild. Both years in Berlin have similar stories, with 2019 showing some minor gusts.

At this time, I'm going to have to write this down as similar to my first outcome. I do not have enough specific data to make specific conclusions about what happens in July in these World Cup competitions.

## Conclusions

I would have loved if this post showed some amazing insight into archery tournaments, showing some hidden dynamic of tournament life that either no one knew, or no one made public. I would have loved if this post showed some significant bump or dip in tournament scores before or after the olympics. But in the end, what this post shows is what happens when we get attached to ideas about our data, and how we should handle the data and ourselves when we are told that what we want to see isn't a part of reality as the data reflects it. 

We should be willing to adapt and understand that all is not as it seems at first glance, regardless of how cool or interesting that first glance was.




Google Colab Notebooks used during this assignment  
[Initial notebook for Olympics exploration and original seasonal bias discovery](https://github.com/VegaSera/World-Archery-Analysis/blob/master/World_Archery_Exploration_and_Arrow_Averages.ipynb)  
[Second notebook for processing Qualifier arrow average, instead of match arrow average](https://github.com/VegaSera/World-Archery-Analysis/blob/master/WA_Arrow_Averages_Qualifiers.ipynb)


