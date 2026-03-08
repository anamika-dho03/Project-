
import pandas as pd
import numpy as np
import ast
     

df = pd.read_csv('International_T20_Data.csv')
     
Rename all column names to appropriate names

df = df.rename(columns={
    'meta.data_version':'data_version',
    'meta.created':'created_date',
    'meta.revision':'revision',
    'info.dates':'match_date',
    'info.gender':'gender',
    'info.match_type':'match_type',
    'info.outcome.by.wickets':'win_by_wickets',
    'info.outcome.by.runs':'win_by_runs',
    'info.outcome.winner':'winner',
    'info.overs':'overs',
    'info.player_of_match':'player_of_match',
    'info.teams':'teams',
    'info.toss.decision':'toss_decision',
    'info.toss.winner':'toss_winner',
    'info.umpires':'umpires',
    'info.venue':'venue',
    'info.city':'city',
    'info.match_type_number':'match_number',
    'info.neutral_venue':'neutral_venue',
    'info.outcome.method':'method',
    'info.outcome.result':'result',
    'info.outcome.eliminator':'eliminator',
    'info.outcome.bowl_out':'bowl_out'
})
     

df.head()
     
innings	data_version	created_date	revision	match_date	gender	match_type	win_by_wickets	winner	overs	...	match_number	neutral_venue	method	result	eliminator	info.supersubs.New Zealand	info.supersubs.South Africa	info.bowl_out	bowl_out	team_pair
0	[{'1st innings': {'team': 'Australia', 'delive...	0.9	2017-02-18	2	[datetime.date(2017, 2, 17)]	male	T20	5.0	Sri Lanka	20	...	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	(Australia, Sri Lanka)
1	[{'1st innings': {'team': 'Australia', 'delive...	0.9	2017-02-19	2	[datetime.date(2017, 2, 19)]	male	T20	2.0	Sri Lanka	20	...	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	(Australia, Sri Lanka)
2	[{'1st innings': {'team': 'Australia', 'delive...	0.9	2017-02-23	1	[datetime.date(2017, 2, 22)]	male	T20	NaN	Australia	20	...	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	(Australia, Sri Lanka)
3	[{'1st innings': {'team': 'Hong Kong', 'delive...	0.9	2016-09-12	1	[datetime.date(2016, 9, 5)]	male	T20	NaN	Hong Kong	20	...	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	(Hong Kong, Ireland)
4	[{'1st innings': {'team': 'Zimbabwe', 'deliver...	0.9	2016-06-19	1	[datetime.date(2016, 6, 18)]	male	T20	NaN	Zimbabwe	20	...	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	NaN	(India, Zimbabwe)
5 rows × 28 columns

Top 3 venues hosting the most matches

top_venues = df['venue'].value_counts().head(3)
print("Top 3 Venues:")
print(top_venues)
     
Top 3 Venues:
venue
Dubai International Cricket Stadium    62
Sheikh Zayed Stadium                   41
Shere Bangla National Stadium          39
Name: count, dtype: int64
Pair of teams who played the most matches

df['teams'] = df['teams'].apply(ast.literal_eval)

df['team_pair'] = df['teams'].apply(lambda x: tuple(sorted(x)))

top_pair = df['team_pair'].value_counts().head(1)

print("\nMost frequent team pair:")
print(top_pair)
     
Most frequent team pair:
team_pair
(Australia, England)    45
Name: count, dtype: int64
Top 5 teams by win percentage

matches_played = {}

for teams in df['teams']:
    for t in teams:
        matches_played[t] = matches_played.get(t,0) + 1

matches_played = pd.Series(matches_played)

# matches won
wins = df['winner'].value_counts()

win_percentage = (wins / matches_played) * 100
top5 = win_percentage.sort_values(ascending=False).head(5)

print("\nTop 5 Teams by Win Percentage:")
print(top5)
     
Top 5 Teams by Win Percentage:
Belgium        100.000000
Spain           83.333333
Germany         76.470588
Namibia         73.529412
Afghanistan     68.000000
dtype: float64
Function to generate scorecard

def get_scorecard(innings_data):

    innings = ast.literal_eval(innings_data)

    scorecards = []

    for inning in innings:
        inning_name = list(inning.keys())[0]
        data = inning[inning_name]

        deliveries = data['deliveries']

        batsman_runs = {}
        bowler_wickets = {}

        for ball in deliveries:
            over = list(ball.keys())[0]
            details = ball[over]

            batsman = details['batsman']
            bowler = details['bowler']

            runs = details['runs']['batsman']

            batsman_runs[batsman] = batsman_runs.get(batsman,0) + runs

            if 'wicket' in details:
                bowler_wickets[bowler] = bowler_wickets.get(bowler,0) + 1

        top_batsmen = pd.DataFrame(
            sorted(batsman_runs.items(), key=lambda x:x[1], reverse=True)[:4],
            columns=['Batsman','Runs']
        )

        top_bowlers = pd.DataFrame(
            sorted(bowler_wickets.items(), key=lambda x:x[1], reverse=True)[:4],
            columns=['Bowler','Wickets']
        )

        scorecard = pd.concat([top_batsmen, top_bowlers], axis=1)
        scorecards.append(scorecard)

    return scorecards
     

match_scorecards = get_scorecard(df['innings'][0])

team1_scorecard = match_scorecards[0]
team2_scorecard = match_scorecards[1]

print(team1_scorecard)
print(team2_scorecard)
     
     Batsman  Runs          Bowler  Wickets
0   AJ Finch    43      SL Malinga        2
1  M Klinger    38  PADLR Sandakan        1
2    TM Head    31   DAS Gunaratne        1
3  AJ Turner    18   JRMVB Sanjaya        1
           Batsman  Runs      Bowler  Wickets
0    DAS Gunaratne    52     A Zampa      2.0
1   EMDY Munaweera    44   AJ Turner      2.0
2      N Dickwella    30  PJ Cummins      1.0
3  TAM Siriwardana    15         NaN      NaN
