# Live_IPL_Dashboard
This dashboard gives a live snapshot of the ongoing IPL Cricket match using data fetched from REST APIs

APIs for IPL matches from website - https://rapidapi.com/
Register an application on RapidAPI website and using the unique RapidAPI key, you can query different APIs to get suitable results.

STEPS TO CREATE LIVE IPL DASHBOARD 
Search for Cricbuzz Cricket on RapidAPI website

To get IPL 2024 matches info:

1. Series > series/list endpoint
	parameter - league
	Expand the JSON response to get the IPL series ID - 7607

2. Series > series/get-matches
	parameter: seriesID - 7607
	Expand the JSON response to get the details about all matches list of IPL 2024

Now, copy the javascript code for get matches from rapidAPI website and use chatgpt to convert it to M language code
> Copy the M Query code and paste it in Power BI - Get Data > Blank Query > Advanced Editor in Home tab

------------------------------------------------------------------------
<CODE> :
let
    // Define the API endpoint URL
    apiUrl = "https://cricbuzz-cricket.p.rapidapi.com/series/v1/7607",

    // Define the headers
    headers = [
        #"X-RapidAPI-Key" = "dc16c8f47cmshc56f781f304cc9ep196618jsn10bb77e50313",
        #"X-RapidAPI-Host" = "cricbuzz-cricket.p.rapidapi.com"
    ],

    // Function to make API request
    getAPIData = (url as text, headers as record) => 
        let
            // Make the request
            response = Web.Contents(url, [Headers=headers]),
            //Parse the JSON response
            jsonResponse = Json.Document(response)

        in
            jsonResponse,
    // Call the function to get API data
    apiData = getAPIData(apiUrl, headers)
in
    apiData

------------------------------------------------------------------------

Optionally you might be asked to specify login credentials to connect > Click on it > specify Web URL(same GET URL is loaded) to get data from web > OK

Records are fetched to power query

Name this Query as 'Base API'
right-click > Reference - 'Matches List'

Select 'Matches List' query and Click on 'Into Table'

Click on List(link) of matchDetails in the results

Click on 'Into Table'

In dialog box -> delimiter - None, handle extra columns - show as errors

Go on expanding the columns and remove unwanted columns

(Expand columns <=> Expand to New Rows)



GET MATCH DETAILS - SCORE

Duplicate the 'Base API' Query - Rename 'Base API Live Score'

Get match ID from previous API results - ex: 91479

RapidAPI website:
matches > matches/get-scorecard 
	parameter 91479
	Test Endpoint

Get the URL from Code Snippets

Open advanced editor of 'Base API Live Score' and replace URL > OK

right-click on 'Base API Live Score' > Reference - 'Live_Match_Score'

Click on 'List' of scoreCard in results > To Table > OK

Expand columns



> TO MAKE CODE DYNAMIC - To get live match details

Make reference of 'Matches List' Query > 'Live Match'

Grouping -> Select 'Base API', 'Matches List' and 'Live Match' queries > right-click > Group > 'IPL Match List'

In 'Live Match' query, filter 'state' column - 'In Progress'

NOTE: If at any point you get " Formula.Firewall: " ERROR, go to File > Options & Settings > Options > CURRENT FILE section > Privacy > select Ignore the privacy levels > OK

Go to 'Base API Live Score' Query > Applied Steps - select apiURL > Home > Advanced Editor

CODE :

	1. Add : matchID = Text.From(Table.First(Live_Match)[matchId])
	2. Replace : apiURL as apiUrl = "https://cricbuzz-cricket.p.rapidapi.com/mcenter/v1/" & matchID & "/scard"

	NOTE : Text.From() function is used to convert matchId in 'Live_Match's matchID column which is of data type Whole Number to Text to be able to concatenate with apiUrl


right-click on 'Base API Live Score' > Reference - 'Live_Batsman_Score'

right-click on 'Base API Live Score' > Reference - 'Live_Bowler_Score'

'Live_Batsman_Score' > Click on List > To Table > Expand Columns > Remove unwanted columns

		> After expanding Batsmen data column, select all other columns except bat1, bat2, ... batn > Unpivot other columns
		> Expand column

Do the same with Bowler query


Import Team_Logo excel data 

Group all Live Match related queries in 1 group - 'Live Match Data'


TO GET TEAM LOGO URL

Go to 'Live_Match_Score' Query and Merge queries with Team_Logo table 
		> first with batTeamName and Team
		> next with bowlTeamName and Team

Expand column and select Team logo (image url link)



TO GET PLAYER IMAGE URL

Import IPL_Player_Summary.csv file

Go to 'Live_Batsman_Score' Query and Merge queries with IPL_Player_Summary table 
with batName and Player  > Expand columns - select Image

Generate online web URLs for batsman and bowler images - https://postimages.org/ website

For the players who have null values in image column -> Replace Values > postimages Direct Link for images (.jpg)
(Also for the .svg values)

Repeat the same procedure for 'Live_Bowler_Score' Query



GET TOSS DETAILS

Create a reference of 'Base API Live Score' > Rename it to 'Toss_Info'

Open Match Header : Record Link > tossResults : Record Link


In 'Live_Match_Score' query, create a custom column, Current score = Text.From([runs]) &"(" & Text.From([overs]) & ")"


Close & Apply


------------------------------------------------------------------------------------------------------------------------------------------------


Create a table for configuration as 'Config'

Add measures :
	> Inning_1 = 1
	> Inning_2 = 2

Create another table for measures as 'Live_Match_Measures'

Add measures :
> Innings1_Batting_Team = var inn = [Inning_1]
RETURN
CALCULATE(VALUES(Live_Match_Score[batTeamName]),KEEPFILTERS(Live_Match_Score[inningsId]=inn))

> Similarly Innings2_Batting_Team

> Innings1_Score = var inn=[Inning_1]
RETURN
CALCULATE(VALUES(Live_Match_Score[Current_score]),KEEPFILTERS(Live_Match_Score[inningsId]=inn))

> Similary Innings2_Score

> Innings1_Runrate = var inn=[Inning_1]
RETURN
CALCULATE(VALUES(Live_Match_Score[runRate]),KEEPFILTERS(Live_Match_Score[inningsId]=inn))

> Similary Innings2_Runrate

> Current_Innings = COUNT(Live_Match_Score[inningsId])

> Live_Score = IF([Current_Innings]=1, [Innings1_Score], [Innings2_Score])

> Live_Batting_Team = IF([Current_Innings]=1, [Innings1_Batting_Team], [Innings2_Batting_Team])

> Innings1_Bowling_Team = var inn = [Inning_1]
RETURN
CALCULATE(VALUES(Live_Match_Score[bowlTeamName]), KEEPFILTERS(Live_Match_Score[inningsId]=inn))

> Similarly Innings2_Bowling_Team

> Live_Bowling_Team = IF([Current_Innings]=1, [Innings1_Bowling_Team], [Innings2_Bowling_Team])

> TossWinner = CALCULATE(VALUES(Toss_Info[Value]), FILTER(Toss_Info, Toss_Info[Name]="tossWinnerName"))

> Toss_Decision = CALCULATE(VALUES(Toss_Info[Value]), FILTER(Toss_Info, Toss_Info[Name]="decision"))

> Toss_Results = 
var teamID = CALCULATE(VALUES(Toss_Info[Value]), Toss_Info[Name]="tossWinnerId")
var teamShortName = CALCULATE(VALUES(Live_Match[team1_SName]), KEEPFILTERS(Live_Match[team1_teamId]=VALUE(teamID)))
var teamName = IF(ISBLANK(teamShortName), CALCULATE(VALUES(Live_Match[team2_SName]), KEEPFILTERS(Live_Match[team2_teamId]=VALUE(teamID))),teamShortName)
RETURN
teamName & " won the toss & chose to go with " & [Toss_Decision]

Edit Team_Logo table columns that were merged to Live_Match_Score and checkbox captain image urls for batting and bowing teams

> Innings1_Batting_Team_Captain_Img = var inn = [Inning_1]
RETURN
CALCULATE(VALUES(Live_Match_Score[Batting_Team_Captain]), KEEPFILTERS(Live_Match_Score[inningsId]=inn))

> Similarly Innings1_Bowling_Team_Captain_Img, Innings2_Bowling_Team_Captain_Img, Innings2_Batting_Team_Captain_Img

> Live_Batting_Team_Captain_Photo = IF([Current_Innings]=1, [Innings1_Batting_Team_Captain_Img], [Innings2_Batting_Team_Captain_Img])

> Live_Bowling_Team_Captain_Photo = IF([Current_Innings]=1, [Innings1_Bowling_Team_Captain_Img], [Innings2_Bowling_Team_Captain_Img])

> Runs_to_win = IF(([Current_Innings]=2), ([Innings1_Score]+1-[Innings2_Score]), 0)

> Req_Run_Rate = VAR curr_overs=CALCULATE(VALUES(Live_Match_Score[overs]),Live_Match_Score[inningsId]=2)
VAR overs_remaining = IF(MOD(((20-curr_overs)*10),10)>0, (19.6-curr_overs), (20-curr_overs))
RETURN "Reqd. Run Rate : " & DIVIDE([Runs_to_win], overs_remaining)



To add IMAGE VISUALS,
Use "Card New"
> Drag the measure > Turn off all display (header, value, background) > Images > Image URL > fX > url will be automatically selected to measure value

LIVE BATSMAN TABLE VISUAL
> Add filter - outDesc = "batting"

OR Create a Calculated table of current playing batsmen - current batting team and outDesc status = "batting"
	> Live_Batsman_Score = VAR curr_bat_team = [Live_Batting_Team]
		RETURN
		FILTER(SUMMARIZE(Batting_Team_Data, 					Batting_Team_Data[batName],Batting_Team_Data[batTeamName],Batting_Team_Data[inningsId],Batting_Team_Data[balls],Batting_Team_Data[boundaries],Batting_Team_Data[fours],Batting_Team_	Data[Image],Batting_Team_Data[mins],Batting_Team_Data[outDesc],Batting_Team_Data[runs],Batting_Team_Data[sixes],Batting_Team_Data[strikeRate]),
	AND(Batting_Team_Data[batTeamName]=curr_bat_team, Batting_Team_Data[outDesc]=[current_batting_status]))

LIVE BATSMAN TABLE VISUAL
> Add filter - inningsId - Top 1 - Max(inningsId)



> Current_Partnership = 
var bat1 = [batsman1]
var bat2 = [batsman2]

var bat1_score = CALCULATE(VALUES(Live_Batsman_Score[runs]), KEEPFILTERS(Live_Batsman_Score[batId]=bat1))
var bat2_score = CALCULATE(VALUES(Live_Batsman_Score[runs]), KEEPFILTERS(Live_Batsman_Score[batId]=bat2))

var bat1_balls = CALCULATE(VALUES(Live_Batsman_Score[balls]), KEEPFILTERS(Live_Batsman_Score[batId]=bat1))
var bat2_balls = CALCULATE(VALUES(Live_Batsman_Score[balls]), KEEPFILTERS(Live_Batsman_Score[batId]=bat2))

RETURN
"Current Partnership : " & (bat1_score + bat2_score) & "(" & (bat1_balls + bat2_balls) & ")"



