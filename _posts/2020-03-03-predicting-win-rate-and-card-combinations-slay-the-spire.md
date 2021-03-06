---
layout: post
title: Slay The Spire - Predictive Modelling and Card Value Analysis
subtitle: Do some cards and relics shine far above others?
published: true
date: '2020-03-03'
---

First, we need to explain some important aspects of what Slay the Spire is, and how it relates to our project.
![Slay The Spire](https://i.imgur.com/RHnA3ep.png)

Slay the Spire is a card-based video game, the goal of which is to ascend the spire and defeat the boss at the end. During the course of the game, the player is presented with many random choices and events, and makes many decisions, which can either make or break their 'run'.

A few things we need to define before we start.
* **Run** - A run is what we define as a single standalone game. The start of each run is typically in the same configuration, although Neow can give different bonuses at the start.
* **Cards** - Cards are the actions that a player can take during combat during the game. Each card comes with an energy cost, and an effect. Cards can also be upgraded to increase their effects, indicated with a '+' next to their name. A few example cards below (Pictures):
  * Strike - 1 cost, deals 6 damage to a target.
  * Defend - 1 cost, blocks 5 damage.
  * Whirlwind+ - X cost, deals 8 damage to all enemies X times. (Uses all energy when played)
  ![Cards](https://i.imgur.com/UZEdXKV.png)
* **Relics** - Relics are items that the player can find during their time in the Spire. These relics cannot be used directly, but their effects are either passive, or can be triggered with certain cards or conditions. Examples of relics:
  * Burning Blood - At the end of combat, heal 6 HP.
  * Ornamental Fan - Every time you play 3 attacks in a single turn, gain 4 Block.
  * Fusion Hammer - Gain 1 energy at the start of each turn. You can no longer Smith at Rest Sites.
  ![Relics](https://i.imgur.com/ycQgSoO.png)
* **Ascension** - When the player successfully wins by defeating the last boss, they are given the option to 'ascend' on future runs. Each ascension makes the game slightly harder in some way, to a max ascension level of 20.
* **Class** - As of time of writing, there are four character classes in the game, each with their own pool of cards and general playstyle. There is the Ironclad, the Silent, the Defect, and the Watcher. For the purposes of this blog post, we will only focus on the Ironclad.
* **Other** - There are some other terms which are common to video games in general, terms like HP (Hit Points), Gold (a currency), etc.

## What are we looking for?
What we are looking for with this project is whether or not we can correctly predict if a run will win, based on the cards, relics, and Ascension level. I would also like to be able to interpret the models to see if there are specific cards or combinations that are more indicative of a winning run.

### Sourcing our data
I sourced this data directly from Megacrit (the developers), who were more than happy to submit this data for analysis and modelling. They sent me 40GB of data, totalling over 50 million runs. This is a bit much to deal with, so we must trim this down a little.

### Cleaning and preparing our data
My first step was to trim the data down to a manageable size. I took only a small subsection of the available data, 600 of the JSON files provided out of nearly 32,000.

Secondly, because the classes are mostly self-contained, I decided that it would be best to model only a single class, but write the code in such a way that any class could be modelled, or indeed all of them. In this instance, I chose the Ironclad. It is the oldest class and perhaps the most well understood by the Slay the Spire community. Next up is our feature selection.

![Columns in the dataset](https://i.imgur.com/ClWfoGI.png)

As you can see, we have a lot of data for each individual run, including a lot of things we dont particularly care about, a lot of which would introduce leakage into our model. Features like floor_reached, playtime, and score are directly correlated to if a player wins or not. The Spire has a set number of floors, runs tend to last for a reasonably consistent amount of time (speedrunners not withstanding), and score tends to go up as the player completes floors. Features like this, while they would certainly increase our model's scores, would do nothing to provide us with any insights to the cards or the relics.

Additionally, there are some features that indicate runs that we would want to exclude entirely. Features like is_seeded or is_endless. To seed a run is to provide it a specific number to it's random generator, which guarantees specific layouts and outcomes, provided you play the same way. Endless runs are unwinnable by default, so we will exclude those also.

![Image of pared down dataset](https://i.imgur.com/tA7JLWj.png)

We get rid of all of the features we dont care about immediately and we're left with three features. We're left with the list of cards that the player finished the run with, the list of relics the player finished the run with, the ascension level the run was on, and whether or not the run won or not. However, we're not yet ready to train our model. Both the master_deck and relics columns are simply just strings, our model can't work with them as is.

So we write a function that takes each string in each cell, removes all of the needless characters like quotation marks and brackets, splits on the commas, and returns a Python List. We do this on both the master_deck column and the relics column. Unfortunately our models still can't deal with it like this, so we need to fix that now. We'll fix this by taking the total contents of the master_deck and relics columns, and effectively One-Hot Encode them. One-Hot Encoding is to take each feature, and assign it either a 0 or a 1, depending on the 'truthiness' of that value. In this case, it's a 1 if the player had a specific card or relic at the end of the run.

We now have gone from four columns to 690 columns, and we're seeing cards like Masterful Stab(A Silent card) and Loop (A Defect card) appearing as well. The culprit? Meet the Prismatic Shard.

![Prismatic Shard](https://i.imgur.com/sfPT0hh.png)

Prismatic Shard allows us to choose cards from other classes when we are offered cards. So at this point, we have two options. We can leave the dataset as is, and deal with our nearly 700 features, or we can remove prismatic shard and all of the extra cards that come with it. For the sake of simplicity and the model, I decided on the second option. Luckily, with everything separated out like we have it, it's trivial to filter by a specific relic.

`df = df[df['RELIC_PrismaticShard'] == 0]`

After which we drop all of our columns that only contain zeroes. This removes all of the cards that only appeared with the help of the Prismatic Shard. This is by no means perfect or comprehensive, as we still have columns like Footwork (A Silent card) that the player must have obtained through other means, but we have reduced our number of features from 695 to 524. We could reduce this further by making a comprehensive list of relics and cards that are unique to a specific class, but I have chosen not to at this time.

After all of this, we're just about ready to begin training models. The only thing that's left is to drop our original master_deck and relics features and cast the victory column as a boolean. Now we can begin.

# Predictive Modelling

To start out, we check our baseline for this set:

[92.5% Accuracy Baseline, assuming all runs fail]

During the various stages of modelling, with different sized subsections of the dataset, we have seen this baseline oscillate between 88% and 93% failure rate. However for this model, we will take the above value as our official baseline.

For every model, we will show the model's permutation importances. Keep in mind that these values show how important the feature is to the model, and does not necessarily dictate which direction the feature might push the result.

## First Model - Random Forest Classifier
Our random forest classifier gives us an accuracy of 92.88% on Validation and 92.99% on the test set. This is only barely above our baseline of 92.5%. 

![RF Permutation Importances](https://i.imgur.com/L2S6W4N.png)



## Second Model - Gradient Boosting with XGBClassifier
Our XGB Boost Classifier gives us an accuracy of 93.3% on Validation, and 93.3% on the Test set. Still just barely above our baseline but slightly improved.

![XGB Classifier Permutation Importances](https://i.imgur.com/7HvJ5x4.png)




## Third Model - Ridge Classifier with RidgeClassifierCV
Ridge Classifier gave us our best results, 94.26% on Validation, 94.0% on the Test set. 

![Ridge Classifier Permutation Importances](https://i.imgur.com/7xagtWs.png)

Our Ridge Classifier much more strongly prioritizes relics over cards, which is a curious result.


# Conclusion
There are a number of conclusions we can draw from this data.

Firstly, looking at all of our permutation importances, we can see that the weight for every feature is rather low. This is to be expected, because having certain cards can slightly help your chances, but ultimately what determines your chance to win is your own skill and random chance. We see that some cards are more important than others, though.

This graph is based off of the shap values of our XGB Classifier, as I could not get this same style graph for the other two models. However the unsorted feature importances for each model tend to agree.
![Shap Importances](https://i.imgur.com/KYjQtis.png)

First and foremost, every card the Ironclad starts with leans more towards losses, the most impactful of which being the basic Strike card. This is something that is corroborated by high level players, who often remove these basic strikes from their decks.

The other starting cards also follow this pattern as well. Bash and Defend, as well as to a lesser extent the upgraded Strike. Bite also has a similar effect, despite not being a starter card.

Most other cards and relics however have a positive impact on win rate. Shrug It Off +1, the Red Mask relic, Heavy Blade +1 and Pommel Strike +1 have very strong impacts on how often that deck wins.

Ascension Level has a negative impact in this model. Higher ascensions tend to result in more losses, which is expected given that ascensions increase the difficulty.

## What more could be done?
I wrote the code with more exploration in mind. We could quite easily run this same code with the Silent and the Defect as well, if we had the time to run it.

We could also get rid of a lot of the 'unimportant' features by specifying which cards/relics belong to which class. This would improve our model training speed by drastically reducing the number of features.

We could filter the shorter runs out to see how much of an effect the base cards have on runs that last long enough to have had the opportunity to get rid of them.

We could also do more hyperparameter tweaking and tuning, which I did not do a huge amount of during this project, and we can also fit other model types for more insights.

### Colab Notebooks and Github Links
[Notebook for this project](https://colab.research.google.com/drive/1zDhfoeg9uJOAC45ZHPGLhceZT6A-Psos)