---
title: Visualizing Clutch Scoring in Spain's Soccer League
subtitle: Scores right before games wrap up in the 2017-2018 season.
image: /img/tom-grimbert-718473-unsplash.jpg
---
High level professional soccer tends to be a low scoring game, many matches are left without any of the teams having scored. Lately this has left me curious to find if soccer does have “clutch” players or even “clutch” teams.

I began parsing through JSON formatted text from an API and came up with a table for all goals scored last season along with a rudimentary calculation for what I thought could be a score to grade each goal’s “clutch” level.

Here’s a peak:

![Clutch DataFrame Snapshot](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/clutch-data.png)

After spending hours cleaning the data, coming up with my little clutch score grading formula, digging deeper into scores and game results I found out that my little experiment would take a lot more time to produce some worthwhile conclusions about who are European soccer’s decisive players when the game is on the line.

In my process initially I looked at the obvious. Who are the highest scoring players? Which players have most assists? Which players score or assist the most in the final minutes? Things which are probably found on ESPN within a few clicks.

Then I dug a little more to find more interesting things like players with most goals that tied a game when their team was losing, break a tie, or score to increase the lead by 2 goals (to seal the win) all within 15 minutes of the end of the match.

<img align="center" src="https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/clutch-players.png" alt="Clutch Players Table]"/>

![Clutch Players Graph](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/clutch-top.png)

Now here is when I really found out that it would take a lot more time than what I had available to concretely find those momentous clutch players. So I dug into the teams to see how if there could be something like a clutch team for the season. Again, I looked at goals that tied, broke ties, or seal a win in the last 15 minutes of each game and found that most of these are not the most popular.

![]()

Theas are mostly lower budget teams (when compared to renown teams of Europe) that fight to win with what they have. Teams that challenge the *dream-team* like squads.

![Clutch goals for all teams](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/clutch-allteams.png){: center-block :}

For future reference I will make note of the teams with higher number of wins by clutch goals. These may very well be the most exciting teams. Those that keep you waiting until the last minute, because anything could happen even when it may seem like it’s over.

--> IMAGE <--

As seen in the graphs above Real Betis rescued 10 games by stepping up in the final minutes of these games. Also Levante, a team picked up the pace towards the end of last season and has been doing well this season did much with 11 clutch goals.

Barcelona, Real Madrid, and even Atletico de Madrid are the leaders in Spain’s top league but teams like Real Betis, Levante and Villarreal (not labeled in graph above) keep the season enjoyable and challenging. For example Levante’s team squad is worth 10 times less than what Barcelona or Real Madrid are worth and Real Betis may have the oldest age average of all teams in the league.

![Teams with Clutch Success](https://firstpythonbucketac60bb97-95e1-43e5-98e6-0ca294ec9aad.s3.us-east-2.amazonaws.com/clutch-success.png){: center-block :}

Although I did not find what I sought after initially (and do aim to take the time to further research as time allows) the work did reward me with the lesson of not coming to data with preconceived ideas and a deeper appreciation for the game of soccer.
