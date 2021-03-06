# Research Goal

To build an NFL draft model capable of producing meaningful player predictions.  I had originally planned to do so using a fuzzy Random Forest trained on NFL Combine and Pro Day physical measurements, individual and team college statistics, and engineered features.  The model produced superior results when treating physical measurements as crisp rather than fuzzy, which was surprising but nonetheless forced me to change my approach.

A Random Forest model is appropriate for this dataset because of the relatively small number of observations (roughly 250-300 players per draft class) and the highly non-linear relationship between the input and output variables.  Random Forests are fairly robust against overfitting, which is a concern when modelling noisy data.

Player performance is impacted by round and team selection in the draft - first-round selections receive more opportunities than seventh-round selections, different schemes fit some players better.  Because of this the model performance can be greatly improved by including some regression to draft selection or, in the case of test data, public rankings.

# Model Output

I've decided to take the novel approach of using player ratings from EA Sports' Madden video game franchise as a proxy for player production, skill, and value.  This is beneficial for a number of reasons.  The first is that these ratings provide continuous output on a consistent scale across both years and positions; a player rated 99 overall is considered to be elite at their position, regardless of the unique responsibilities or challenges in quantifying performance specific to that position.  The second reason is that Madden ratings predate modern quantitative evaluative metrics like those provided by Football Outsiders or Pro Football Focus. 

Madden ratings explained - https://fivethirtyeight.com/features/madden/#

Overall ratings are calculated using position-specific formulas that weight individual attributes like speed, strength, and tackling.  Ratings are updated each year through a Bayesian-like process of weighing new information to update old.  To aggregate ratings for each player, I use a 5-part mean which includes ratings in Years 1-4 and Peak rating. 

NFL rookie contract length is 4 seasons (along with a fifth year club option for first-round picks), while the average career length in the NFL is less than 4 years.  As such, when building a draft model is makes sense to only consider production accrued during the first 4 years of a player's career.

Year 1 represents the Madden rating given to each player following their rookie season.  For this reason, the final year for which complete data is available is the 2014 draft class (with Madden 19 providing Year 4 ratings).  This decision was made to better capture NFL success, as rookie player ratings are highly dependent on draft order.  For example, in Madden 2008 rookie #1 overall pick JaMarcus Russell was awarded an overall rating of 82, just 1 point lower than #3 overall pick Joe Thomas.  The next year, Russell's rating was 83, while Thomas was a 97 overall.  By Madden 2010, Russell was given a rating of 72 overall, while Thomas maintained his 97 overall rating.  Year 3 and Year 4 ratings have been given double weight for the same reason, with the added effect of lowering the ratings of players who were not able to stay in the league for at least 4 years.

While this metric on the whole does a good job of ranking player talent and production, it is blind to players who peaked later in their careers or those who had short careers.  Notable examples of each include Eric Weddle (84.6 rating, eventual 2x All Pro, 6x Pro Bowl) and Jon Beason (95.4 rating, 1x All-Pro, 3x Pro Bowl).  Weddle did not reach his peak until after re-signing with the Chargers as an unrestricted free agent prior to the 2011 season, and could have presumably reached his peak while playing for another team.  Beason suffered an Achilles injury during the 2011 season and eventually lost his job with the Panthers, starting in only 26 games in the years following his rating window.  Beason would have been eligible to sign as a free agent following the 2011 season had the Panthers not offered a contract extension.  

In the NFL, the drafting team maintains the exclusive right to employ each player for 4 years following their selection, thus it is incumbent upon the team to select and develop players who provide the most value during that period.  For that reason I stand by the decision to evaluate draft selections only on a player's first 4 years in the league.  

# Dataset

I wrote several web scraping programs to pull data from NFL Draft Scout (an excellent resource for Combine data, and the only source I'm aware of that includes Pro Day data), Pro-Football-Reference, and CFB Reference (both Sports-Reference-operated sites, easily the best sources for football statistics in the NFL or FBS).

The dataset covers the 2006-2014 draft classes and includes players who were ranked in NFL Draft Scout's top 300 in their draft year.  I have removed all quarterbacks, kickers, punters, long snappers, and fullbacks due to the relatively small sample sizes or extreme specialization that each position requires.  It might be valuable to evaluate these positions later – particularly quarterbacks – but for now the model focuses exclusively on 13 "skill" positions, bucketed into 7 position groups.

The dataset restrictions exclude some notable players ranked outside of the top 300, both drafted and undrafted, who went on to varying degrees of success in the NFL.  At the top extreme are 4-time All Pro Antonio Brown and Super Bowl LIII MVP Julian Edelman.  But while many players on this list never played a down in the NFL, it is important to be aware of which players are excluded and it may be worthwhile to expand the dataset in the future.

I have removed players from the dataset whose NFL careers were cut prematurely short either voluntarily or involuntarily (due to injury, not ability).  These players' ratings (or lack thereof) are not representative of their production and thus only serve to complicate the dataset and confuse any modeling attempts.  Examples include Aaron Hernandez, Gaines Adams, and Chris Borland.  The list is as long as it is depressing.

There is also a subset of players who drastically changed position upon entering the league.  This is contrary to less extreme position changes (tackle to guard, cornerback to safety), which occur frequently.  These players have been removed because their college statistics create noisy data.  Examples: Denard Robinson, Devin Hester, J.R. Sweezy.

# College Statistics

College statistics have been collected and cleaned at the FBS level from Sports Reference.  Using college statistics is important because they provide information on a player's in-game performance.  However, college football styles vary greatly among teams and have changed over time.  Therefore we must control for differences in pace and style of play when considering college numbers.  Rather than attempt to fit a model on raw season total statistics, I've decided to use neutralized per game statistics under the following parameters:

-	Receiving statistics have been normalized to 30 team pass attempts per game
-	Rushing statistics have been normalized to 35 team rush attempts per game
-	Sacks/Interceptions/Passes Defended normalized to 30 opponent pass attempts per game
-	Tackles/TFL/Fumbles normalized to 65 opponent offensive plays per game
-	Regression to position-specific mean as function of percentage of team games played 

To illustrate this point let's look at Calvin Johnson and Michael Crabtree, who were both highly productive college wide receivers selected early in the first round.

-	Calvin Johnson, Georgia Tech (2006): 14 G, 76 catches, 1202 yards, 15 TD
-	Michael Crabtree, Texas Tech (2008): 13 G, 97 catches, 1165 yards, 19 TD

The two statlines appear very similar without context.  It's easy to make this distinction empirically, but little effort has been made to translate college statistics into more informative data.  Johnson and Crabtree put up similar overall numbers, but Crabtree did it in an air raid style offense that relied heavily on passing while Johnson played on a more balanced offense.

-	Georgia Tech (2006): 368 total team pass attempts (26 per game)
-	Texas Tech (2008): 662 total team pass attempts (51 per game)

When we neutralize both players' statistics, we can better compare each player's level of production.

-	Calvin Johnson (neutralized): 6.2 catches, 97.6 yards, 1.2 TD
-	Michael Crabtree (neutralized): 4.4 catches, 52.8 yards, 0.9 TD

Compare those numbers to each player's NFL career statistics:

-	Calvin Johnson (NFL per game): 5.4 catches, 86.1 yards, 0.6 TD
-	Michael Crabtree (NFL per game): 4.5 catches, 53.0 yards, 0.4 TD

This is a cherry-picked example but it does well to show that while raw statistics are not to be trusted, college data when put into the proper context can be made more predictive.  On a larger scale, we can compare RMSE of the model when including raw college statistics compared to pace- and schedule-neutralized statistics.  Controlling for strength of schedule does not improve the predictiveness of the model, but controlling for pace and style of play does have a significant effect.

| Neutralization | RMSE |
| :---: | :---: |
| Raw per Game | 8.065 |
| Pace-Neutralized per Game | 8.029 |
| Pace- and Schedule-Neutralized per Game | 8.058 |

Here's the full stat list, with a few notable performers:

**Offensive Statistics**
- Rushing Attempts (Andre Williams, Kevin Smith)
- Rushing Yards (Matt Forte, Ray Rice)
- Rushing TD (Toby Gerhart, Ahmad Bradshaw)
- Receptions (Greg Jennings, Zach Ertz)
- Receiving Yards (Hakeem Nicks, Jarvis Landry)
- Receiving TD (Demaryius Thomas, DeAndre Hopkins)
- Scrimmage Yards
- Total Offensive TD
- Non-Offensive TD (C.J. Spiller, Maurice Jones-Drew)

**Defensive Statistics**
- Tackles (Luke Kuechly, Patrick Willis)
- Run Stuffs (Navorro Bowman, Aaron Donald)
- Sacks (Elvis Dumervil, Whitney Mercilus)
- Total TFL (Ryan Kerrigan, Brandon Graham)
- INT (Eric Weddle, Earl Thomas)
- Passes Defended (Dee Milliner, J.J. Watt)
- Fumbles Forced
- Fumbles Recovered
- Non-Offensive TD (Tyrann Mathieu, Darrelle Revis)


# NFL Combine and Pro Day Measurements

The final major inputs of the draft model are the physical measurements taken at the NFL Combine and university Pro Days.  Pro Day measurements are harder to come by due to their decentralized and often scarcely reported nature.  Fortunately, NFL Draft Scout has maintained a database of reported Pro Day measurements spanning the years in our dataset.

There is an enormous benefit in using Pro Day measurements in a model like this.  It allows for a larger training set by including data on players who were not invited to the NFL Combine, but also provides much more complete data because not all players who attend the combine perform the full slate of workouts.  This lessens the need for imputation and reduces uncertainty.

However, there is bias observed in Pro Day measurements.  Pro Days are typically scheduled in the weeks following the NFL Combine, giving players more time to train for the specific physical events.  Furthermore, they often take place at the players' home campuses in environments in which the players feel more comfortable.  Lastly, many events (most notably the 40-yard dash) are hand-timed at Pro Days, leading to better reported times than the electronic times at the Combine.  Each of these factors contributes to improvement in every event among the population of players who participated both at the NFL Combine and at their university Pro Day.

**Players who participated in both NFL Combine and Pro Day**

| Measurement |	Combine |	Pro Day |	n |	Sigma |	Adjustment |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 40 Yard Dash |	4.80 |	4.70 |	831 |	0.076	| + 0.07 |
| 20 Yard Split |	2.79 |	2.71 |	733 |	0.065	| + 0.06 |
| 10 Yard Split	| 1.68 |	1.62 |	739 |	0.057	| + 0.04 |
| Bench Press |	20.0 reps |	21.7 reps |	254 |	2.556	| - 1.2 |
| Vertical Jump |	31.7" |	33.6"	| 593	| 2.342	| - 1.3" |
| Broad Jump |	112.9" |	115.2" |	481	| 4.356	| - 1.6" |
| 20 Yard Shuttle |	4.46	| 4.42 |	424	| 0.155	| + 0.03 |
| 3 Cone Drill |	7.34 |	7.22 |	342	| 0.223	| + 0.08 |

In order to correct for this bias, I've (somewhat arbitrarily) chosen to shift recorded Pro Day measurements by 70% of the mean delta.  Even when we correct for some of the systematic bias observed in Pro Day measurements, we must also recognize that most physical measurements aren't static.  Some players aren't performing at maximum physical capacity on the day of the Combine<sup>1</sup>, occasionally players injure themselves during their workout<sup>2</sup>, and the measurements aren't always recorded with perfect accuracy or consistency<sup>3</sup>.  

<sup>1</sup> https://www.cleveland.com/osu/2018/03/ohio_state_defensive_end_tyqua_1.html

<sup>2</sup> https://www.washingtontimes.com/news/2019/mar/3/dexter-lawrence-clemson-prospect-injures-hamstring/

<sup>3</sup> https://www.zybeksports.com/hand-timed-versus-electronic-timed-40-yard-dash/

A dataset with this much uncertainty lends itself well to fuzzy set theory.  In simple terms, this will allow us to consider not only a player's recorded 40 yard time of 4.40, but will also consider some probability that their "true" speed is 4.39 or 4.43.  So when the model attempts to predict NFL success given a player's 40 yard dash time, it's not based on a singular number but rather a distribution of times centered around that number.

Fuzzy Set Theory explanation - https://www.doc.ic.ac.uk/~nd/surprise_96/journal/vol4/sbaa/report.fuzzysets.html

My approach is to generate a random forest model on the discrete data, then fit n iterations on randomly shuffled data to generate a distribution of outcomes for each player.  This "shuffling" will occur randomly for each measurement using a normal distribution centered around the discrete number, with sigma equal to half of the standard deviations recorded above. 

In a single random forest, data is crisply split by decision trees based on discrete information.  But with enough randomly shuffled iterations, the trees are no longer binary decisions but rather probabilistic ones centered on each measurement's distribution.  This is particularly relevant for players who may have measurements near decision tree boundaries.  Two players with sprint times separated by mere hundredths of a second are not appreciably different in speed, but a random forest might classify them as such.  The purpose of shuffling is not to fundamentally change each player's physical characteristics, rather to acknowledge measurement uncertainty.  My belief is that this will improve the model outputs over a large enough number of trials.

We have a wealth of NFL Combine and Pro Day data but not every player has participated in every drill, so we'll need to fill in missing values.  Because many of these physical measurements are correlated and most football positions require some degree physical specialization (size, speed, etc.), I've chosen a k nearest neighbor imputation method.  The belief is that if Players A and B are similar in terms of position, size, speed, and quickness, then the two players will also have similar strength or jumping ability.  The exceptions are draft age and wingspan, which can be reasonably predicted using population means.

![alt text](https://i.imgur.com/Qd6gLQ0.png)


# Engineered Features

Perhaps the most essential component of a machine learning model is feature engineering.  

Modern feeling toward physical measurements taken at the Combine is highly dubious, and I agree that each measurement taken in isolation cannot alone adequately define athleticism, much less predict success.  However, there exist more complex metrics which can better perform both tasks across a large enough sample.  

**Body Mass Index (BMI)**
- Body weight (kg) divided by squared height (meters); an attempt to measure the relative amount of mass in an individual


**Speed Score**
- https://www.footballoutsiders.com/stat-analysis/2018/speed-score-2018
- An attempt to contextualize speed, rewards heavier players, all else equal
- *Top performers*: Vernon Davis, Calvin Johnson


**Height-Adjusted Speed Score**
- http://moneyinthebananastand.com/2012/04/24/dominator-rating-height-adjusted-speed-score-and-wr-draft-rankings/
- An attempt to improve upon Speed Score, rewards taller players, all else equal
- *Top performers*: Jadeveon Clowney, Lane Johnson


**Vertical Jump Power**
- https://www.topendsports.com/testing/vertical-jump-power.htm
- An attempt to measure lower body force generation using Sayers Power formula
- *Top performers*: Mario Williams, Ndamukong Suh


**Broad Jump Power**
- Same as Vertical Jump power, using Broad Jump
- *Top performers*: Jamie Collins, Calvin Johnson

 
**Quickness Score**
- Same as Speed Score, replacing 40-Yard Dash time with mean of 10-Yard Split, 20-Yard Shuttle, and 3 Cone Drill
- *Top performers*: J.J. Watt, A.J. Hawk


**Weight-Adjusted Bench**
- 225 * bench press reps, divided by body weight, then adjusted by arm length; rewards player with longer arms, all else equal
- *Top performers*: Stephen Paea, Chris Houston


**Catch Radius**
- 3-dimensional space occupied by the player.  Dimension 1: height, vertical jump, arm length.  Dimension 2: broad jump, distance covered at top speed.  Dimension 3: wingspan, lateral quickness
- *Top performers*: Julio Jones, Antonio Cromartie

The models also include several features designed to summarize the collection of college statistics being used.

**Offensive Usage**
- Rushing attempts plus receptions.  On a neutralized scale, this captures the same information as market share.  An attempt to quantify how involved a player was in their college offense
- *Top performers*: Percy Harvin, Matt Forte

**Defensive Disruption**
- Defensive equivalent to usage, an attempt to quantify how active each defensive player was.  The thought is that better players will make more plays
- *Top performers*: Aaron Donald, Luke Kuechly

**S&P Market Share**
- Combines defensive disruption with team defensive S&P+.  Simply accruing defensive statistics doesn't tell whether the player was effective, so this is an attempt to link activity with effectiveness.  Rewards players who played on better teams, all else equal
- *Top performers*: Ndamukong Suh, Dont'a Hightower


# Cross-Validation and Tuning

I've tuned the model using stratified k-fold cross validation, leaving out each draft class as OOB observations.  As a result, every player has been included in both the training and validation sets.  Each position group has been fit with its own unique hyperparameters to optimize predictions.

**Hyperparameters by Position Group**

| Position | Number of Trees | Max Depth | Max Features | Min Leaf Samples |
| :---: | :---: | :---: | :---: | :---: |
| WR | 100 | 5 | 10 | 3 |
| FS | 250 | 5 | 10 | 1 |
| CB | 50 | 10 | 20 | 2 |
| SS | 40 | 10 | 10 | 2 |
| ILB | 30 | 15 | 5 | 2 |
| RB | 40 | 10 | 10 | 1 |
| TE | 20 | 10 | 3 | 2 |
| EDGE LB | 50 | 5 | 10 | 2 |
| EDGE DL | 250 | 15 | 10 | 2 |
| C | 50 | 5 | 5 | 3 |
| DT | 100 | 3 | 10 | 1 |
| OT | 20 | 5 | 3 | 2 |
| OG | 20 | 10 | 5 | 2 |

Additionally, the model performed best when aggregating predictions from 3 randomized sets, as shown in the plot below.  However, this fuzzy approach failed to outperform discrete features during cross-validation.  I expected the opposite, but it seems treating each measurement as precise leads to the best fit.

![alt_text](https://i.imgur.com/U4BxaGc.png)

**RMSE Using Various Methods**

| Method | RMSE |
| :---: | :---: |
| Discrete | 8.027 |
| 1 Random set | 8.122 |
| 2 Random sets | 8.069 |
| 3 Random sets | 8.063 |
| 5 Random sets | 8.098 |
| 10 Random sets | 8.115 |


# Results and Further Research #

Model outputs from validation can be viewed here: https://docs.google.com/spreadsheets/d/1-ooQ4UTafyFOTWDtbYGmPgdHfspY8bci45tUS6I5-LU/edit?usp=sharing

By and large the model does surprisingly well considering the lack of more traditional evaluative inputs.  NFL teams have the resources of scouting departments providing more detailed player evaluation, experienced coaching staffs evaluating personnel fits, and front offices to balance financial considerations and positional value.  Each of these factor into draft decisions and improve ranking methods beyond the scope of this model.

**Model results by position**

| Position | RMSE | n | Most Important Features |
| :---: | :---: | :---: | :---: |
| WR | 7.523 | 314 | Underclassman, Usage, Age, Srimmage Yards, Total TD, Receiving Yards |
| FS | 7.621 | 128 | S&P Share, Age, SOS, 20-Yard Shuttle, Height, 40-Yard Dash |
| CB | 7.604 | 292 | Age, Quickness Score, Height-Adjusted Speed Score, S&P Share, Height, 20-Yard Dash |
| SS | 7.437 | 107 | Run Stuffs, BMI, Defensive Disruption, Quickness Score, Tackles, TFL |
| ILB | 7.649 | 291 | S&P Share, Age, Tackles, Height-Adjusted Speed Score, Vert Power |
| RB | 8.121 | 194 | Rush TD, Rush Yards, Age, Total TD, Scrimmage Yards, Height-Adjusted Speed Score |
| TE | 8.097 | 139 | Scrimmage Yards, Receiving TD, Receiving Yards, Offensive Usage, Hand Size |
| EDGE LB | 8.846 | 90 | S&P Share, Disruption, Catch Radius, Tackles, Weight, Age |
| EDGE DL | 7.445 | 190 | Age, TFL, Weight, Height-Adjusted Speed Score, Quickness Score, Underclassman |
| C | 8.948 | 82 | 3-Cone, Weight, Broad Jump, Quickness Score, Hand Size, Age |
| DT | 7.823 | 211 | S&P Share, TFL, Tackles, Disruption, Run Stuffs, 3 Cone |
| OT | 8.882 | 222 | Age, Vert Power, Arm Length, Speed Score, Weight |
| OG | 8.819 | 140 | Adjusted Bench, 20-Yard Dash, Catch Radius, Quickness Score, Age, Weight |

When properly optimized, the model can achieve RMSE below 8 during cross-validation. Unsurprisingly, it struggles most with offensive linemen, who lack individual statistics.  In particular it struggles with centers, whose responsibilities in the NFL are as much mental as physical.  Interestingly, NFL teams have had great success evaluating centers, as 4 of the 5 first rounders were named to All-Pro teams in their careers, and all made the Pro Bowl at some point.

As mentioned in the introduction, the model could be improved substantially by including draft selection or consensus rankings.  Furthermore, team-specific random effects could likely explain some of the residuals.  I may eventually explore these research questions, but my short-term priorities are on visualization and presentation of data. 

Album of select draft prospect profiles: https://imgur.com/a/SCdkLj1



# Updated Results and Test Data #

I received good feedback from this Reddit thread: https://www.reddit.com/r/nfl/comments/b27abi/oc_building_an_nfl_draft_model_using_machine/, and have used it to make a few small but impactful changes to the model.  The first is to restrict feature inclusion to limit the risk of spurious results.  It doesn't make sense to include tackles for offensive players, nor rushing yards for defensive players.  It's best to remove these entirely so that the model can focus on meaningful splits.

The second change is to expand the training set for each position to include other positions outside of that group.  This is most obvious in the case of positions without a clear distinction, such as most EDGE players.  In order to keep predictions and interpretations position-specific, I've ensured that the focus position remains the most populous position group within every set of training data.  It turns out that this strategy beneficial for every single position group, while also creating more robust training samples to be used on test data.  The model still struggles relatively with offensive linemen, but the improvement is seen across the board.

| Position | Position Comps | Old RMSE | New RMSE |
| :---: | :---: | :---: | :---: |
| WR | RB, TE, CB | 7.523 | 7.170 |
| FS | CB, SS, RB | 7.621 | 7.293 |
| CB | FS, WR | 7.604 | 7.269 |
| SS | FS, ILB | 7.437 | 7.390 |
| ILB | EDGE LB, SS, WR | 7.649 | 7.326 |
| RB | WR, ILB | 8.121 | 7.725 |
| TE | WR, OT | 8.097 | 7.794 |
| EDGE LB | EDGE DL, ILB | 8.846 | 8.318 |
| EDGE DL | DT, EDGE LB, TE | 7.445 | 7.416 |
| C | OG | 8.948 | 8.637 |
| DT | EDGE DL, EDGE LB, ILB | 7.823 | 7.625 |
| OT | OG, C, EDGE DL | 8.882 | 8.492 |
| OG | OT, C | 8.819 | 8.456 |

Moving on to the good stuff, here's a breakdown of a few notable players most impacted by the changes:

| Player | Position | Change |
| :---: | :---: | :---: |
| Justin Durant | ILB | + 10.7 |
| Chandler Jones | EDGE LB | + 8.5 |
| Maurkice Pouncey | C | + 8.1 |
| Eric Ebron | TE | + 7.8 |
| Jadeveon Clowney | EDGE DL | + 7.2 |
| Marshawn Lynch | RB | + 6.8 |
| Tyson Alualu | DT | - 5.0 |
| Stephen Hill | WR | - 5.3 |
| Earl Mitchell | DT | - 6.1 |
| Kenneth Darby | RB | - 7.5 |
| James Hanna | TE | - 7.7 |
| Dennis Pitta | TE | - 10.2 |

There aren't any easily defined trends in the delta between the two models.  Generally speaking, however, including multiple position groups in the training data allows the model to better handle outliers in test data.  

My next step was to throw unobserved data at the model to see how it would respond.  So far, I've used the 2016, 2017, and 2019 draft classes and am quite pleased with the results.  Consider the top 5 ranked players from 2016:

- 1. Corey Coleman, selected #15 overall, didn't quite work out
- 2. Jalen Ramsey, selected #5 overall, 1x All Pro, 2x Pro Bowl
- 3. Ezekiel Elliott, selected #4 overall, 1x All Pro, 2x Pro Bowl
- 4. Derrick Henry, selected #45 overall, 1000 rushing yards in 2018
- 5. Joey Bosa, selected #3 overall, 1x Pro Bowl

And 2017:

- 1. T.J. Watt, selected #30 overall, 1x Pro Bowl
- 2. Christian McCaffrey, selected #8 overall, 1900 scrimmage yards in 2018
- 3. Curtis Samuel, #40 overall, limited by injury but talented when he plays
- 4. John Ross, #9 overall, complete dud so far
- 5. D'Onta Foreman, #89 overall, looked good before blowing out his Achilles

And 2019:

- 1. Quinnen Williams, projected Top 5 pick
- 2. T.J. Hockenson, projected Top 15 pick
- 3. Devin White, projected Top 10 pick
- 4. NKeal Harry, projected First Round pick
- 5. Maxx Crosby, projected 3rd/4th Round pick


The full test outputs and updated validation sets can be viewed here: https://docs.google.com/spreadsheets/d/1-ooQ4UTafyFOTWDtbYGmPgdHfspY8bci45tUS6I5-LU/edit?usp=sharing

A gallery of new player dashboards can be viewed here: https://imgur.com/a/acAFRt2






