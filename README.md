# NBA Lineups(Data manipulation)
In this project I will try to establish lineup with the highest average scored points/points allowed/points differ/any other basic box score stats  per 24 min in NBA 2016-2019 seasons.
# Data
I use NBA play by play data, where each row represents one event on the court. So for one game there are around 500-600 rows and for whole season around 500k rows. The number of columns is 111. We need to find out way to count stats for each lineup so it will be great project to test data manipulation skills.
# Creating column with lineup data
After identifying the relevant columns and removing the unnecessary ones I create column thats contains indexes of five players in lineup for home teams (I sorted them in case of 
same players in lineup but with mixed postions). Next I did the same for away teams
```r
df_pbp['lineup_h'] = df_pbp.apply(lambda x: "_".join(sorted(
    x[['HOME_PLAYER_ID_1', 'HOME_PLAYER_ID_2', 'HOME_PLAYER_ID_3', 'HOME_PLAYER_ID_4', 'HOME_PLAYER_ID_5']].astype(
        str).to_list())), axis=1)

df_pbp['lineup_a'] = df_pbp.apply(lambda x: "_".join(sorted(
    x[['AWAY_PLAYER_ID_1', 'AWAY_PLAYER_ID_2', 'AWAY_PLAYER_ID_3', 'AWAY_PLAYER_ID_4', 'AWAY_PLAYER_ID_5']].astype(
        str).to_list())), axis=1)
```
# Creating column that will help us count minutes played by each lineup
I created column that contains previous values from colmun 'TIME' and fill empty values with 0(first row, evry match starts with zeros in two frist rows so zero in frist row is justified. Next I count time per event so when I create new dataframe with lineups with no divison into home and away lineups I can group by lineups and sum other stats.
```r
df_pbp['prev_time'] = df_pbp['TIME'].shift(1)
df_pbp['prev_time'] = df_pbp['prev_time'].fillna(0)

df_pbp['time_per_action'] = df_pbp['TIME'] - df_pbp['prev_time']
df_pbp['time_per_action'] = np.where(df_pbp['time_per_action'] < 0, 0, df_pbp['time_per_action'])
```
# Define function to couns stats
Next I define function identifying if someone from the lineup get steal for example. If it happens there will be a 1 in a row otherwise 0. And using this function i can do this for every other stat except points.
```r
def statsCounter(stat_name, new_column_name, if_team_stat, if_home):
    if (if_home == True):
        df_pbp[stat_name] = df_pbp[stat_name].fillna(0)
        df_pbp[stat_name] = df_pbp[stat_name].astype(str)
        df_pbp[stat_name] = df_pbp[stat_name].apply(lambda x: x[:-2])
        if (if_team_stat == True):
            df_pbp[new_column_name] = np.where((df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_5'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_4'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_3'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_2'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_1'])
                                               | (df_pbp[stat_name] == df_pbp['PLAYER1_ID'])
                                               , 1, 0)
        else:
            df_pbp[new_column_name] = np.where((df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_5'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_4'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_3'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_2'])
                                               | (df_pbp[stat_name] == df_pbp['HOME_PLAYER_ID_1'])
                                               , 1, 0)
    else:
        if (if_team_stat == True):
            df_pbp[new_column_name] = np.where((df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_5'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_4'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_3'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_2'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_1'])
                                               | (df_pbp[stat_name] == df_pbp['PLAYER1_ID'])
                                               , 1, 0)
        else:
            df_pbp[new_column_name] = np.where((df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_5'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_4'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_3'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_2'])
                                               | (df_pbp[stat_name] == df_pbp['AWAY_PLAYER_ID_1'])
                                               , 1, 0)

```
# Creating column that will help us count points/2-pointers/3-pointers/freethrows made by each lineup
It will be similar to creating coulmn 'time_per_action' but we have to do this for home and away teams plus count 3-pointers made, 2-pointers made and freethrows made.
Points allowed will be points scored by away team.

```r
df_pbp['prev_home_scored'] = df_pbp['HOME_SCORE'].shift(1)
df_pbp['prev_home_scored'] = df_pbp['prev_home_scored'].fillna(0)

df_pbp['scored_h'] = df_pbp['HOME_SCORE'] - df_pbp['prev_home_scored']
df_pbp['scored_h'] = np.where(df_pbp['scored_h'] < 0, 0, df_pbp['scored_h'])

df_pbp['prev_away_scored'] = df_pbp['AWAY_SCORE'].shift(1)
df_pbp['prev_away_scored'] = df_pbp['prev_away_scored'].fillna(0)

df_pbp['scored_a'] = df_pbp['AWAY_SCORE'] - df_pbp['prev_away_scored']
df_pbp['scored_a'] = np.where(df_pbp['scored_a'] < 0, 0, df_pbp['scored_a'])

df_pbp['3_pointers_made_h'] = np.where(df_pbp['scored_h'] == 3, 1, 0)
df_pbp['2_pointers_made_h'] = np.where(df_pbp['scored_h'] == 2, 1, 0)
df_pbp['3_pointers_made_a'] = np.where(df_pbp['scored_a'] == 3, 1, 0)
df_pbp['2_pointers_made_a'] = np.where(df_pbp['scored_a'] == 2, 1, 0)
df_pbp['free_throws_made_h'] = np.where(df_pbp['scored_h'] == 1, 1, 0)
df_pbp['free_throws_made_a'] = np.where(df_pbp['scored_a'] == 1, 1, 0)

```
# Creating datafarame with players names and players indexes
Based on columns 'PLAYER1_ID' and 'PLAYER1_NAME' i created dataframe that will help us change id's to real player names in our dataframe with lineup and stats. In this columns as the name says we can find id and real name for players that was responisble for action, thats guarante that I will get every player real name related to id. There will be a lot of duplicates so we should drop them and also drop na values if any happen. Next I get players indexes back to separate columns and using map functions i get real names in lineup dataframe.
```r
df_id = df_pbp[['PLAYER1_ID', 'PLAYER1_NAME']]

df_id = df_id.drop_duplicates()
df_id = df_id.dropna()

df_lineups_grouped[['first_player','second_player','third_player','foruth_player','fifth_player']] = df_lineups_grouped.lineup.str.split("_",expand=True)

df_id[['PLAYER1_ID']] = \
 df_id[['PLAYER1_ID']].astype(str)

m = df_id.drop_duplicates('PLAYER1_ID').set_index("PLAYER1_ID")["PLAYER1_NAME"]

df_lineups_grouped.first_player = df_lineups_grouped.first_player.map(m)
df_lineups_grouped.second_player = df_lineups_grouped.second_player.map(m)
df_lineups_grouped.third_player = df_lineups_grouped.third_player.map(m)
df_lineups_grouped.foruth_player = df_lineups_grouped.foruth_player.map(m)
df_lineups_grouped.fifth_player = df_lineups_grouped.fifth_player.map(m)
```
# Grouping the data
After merging home and away lineups and stats  in the way that they now into same columns I use gruopby on 'lineup' column and sum all stats. I divide column 'minutes' by 60 becouse values were in seconds. I also filter for lineups that play more than 100 minutes together. 
```r
df_lineups_grouped = df_results.groupby(by=["lineup"]).sum()
df_lineups_grouped['minutes'] = round(df_lineups_grouped['minutes'] / 60, 2)
df_lineups_grouped = df_lineups_grouped[df_lineups_grouped.minutes >= 100]
```
# Count average stats per 24 min
After some more manipulations i divde every column by column 'minutes' and multiply by 24.
```r
df_lineups_grouped_avg_per_24 = df_lineups_grouped.iloc[:,2:].div(df_lineups_grouped.minutes, axis=0)
df_lineups_grouped_avg_per_24 = round(df_lineups_grouped_avg_per_24 * 24, 1)
```
# Creating points differ column and showing the results
```r
df_lineups_grouped_avg_per_24.insert(11, 'points_differ', df_lineups_grouped_avg_per_24['points_scored'] - df_lineups_grouped_avg_per_24['points_allowed'])
df_top_25_points = df_lineups_grouped_avg_per_24.sort_values(by=['points_differ'], ascending=False).head(25)
```
# Creating dataframe to count players occurecens in top 25 lineups
```r
df_first_player = df_top_25_points_differ['first_player']
df_second_player = df_top_25_points_differ['second_player']
df_third_player = df_top_25_points_differ['third_player']
df_fourth_player = df_top_25_points_differ['foruth_player']
df_fifth_player = df_top_25_points_differ['fifth_player']
frames = [df_first_player, df_second_player, df_third_player, df_fourth_player, df_fifth_player]
series_players_count = pd.concat(frames, ignore_index=True)
df_players_count = series_players_count.to_frame()
df_players_count[0].value_counts()
```
# Results
By changing column in sort_values method we can explore what we want to know for example the best average points differ in 2016-2019 seasons equals 17 and players in this lineup are Amir Johnson, Ben Simmons, Marco Belinelli, Robert Covington and Dario Sarić so it's one of the lineups from Philadelphia 76ers. So a lineup without probably biggest star Joel Embiid. Only 23 lineups reach avg points differ equal or better than 10. In the 25 best lineups according to avg points differ players thats have the most occurences are: Sthepen Curry, Jonas Valanciunas, Klay Thompson, Kyle Lowry, DeMar DeRozan, Jimmy Butler, Draymond Green. Jimmy Butler is the only player that have 3 occurences and every lineup is from diffrent team (Chicago, Phila, Minnesota)
