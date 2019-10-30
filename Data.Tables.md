

Data.Tables
========================================================
author: Mats Hester
date: 27.10.2019
autosize: true
font-import: https://fonts.googleapis.com/css?family=Open+Sans&display=swap
font-family: 'Open Sans', sans-serif;


Data preperation/manipulation
========================================================
incremental: false

Data preparation (transformation / manipulation) is an important step within the data analytics
and in general in the whole data science (crips_dm) process.

- Data scientists spend 80% of their time on colleting, cleaning and organizing data.

- 76% of data scientists view data preparation as the least enjoyable part of their work.

[Source](https://www.forbes.com/sites/gilpress/2016/03/23/data-preparation-most-time-consuming-least-enjoyable-data-science-task-survey-says/#1761fb906f63)

***

![Crisp-DM](CRISP-DM_Process_Diagram1.png)  
Cross industry standard process - data mining

Basics
====================================
Data.Tables is a powerful library for data preperation.
The main goal is to make the process overall faster and use the resources more efficiently than similar libraries like dplyr or python.pandas.

## Advantages
- Compact code
- Speed -> Fread, Fwrite, Joins, 
- Memory efficiency -> no duplicates

***

## Disadvantages
- Unintelligible syntax ->  partly because of the compact code.
- (No duplicates)

## Benchmark

[![Benchmark](Benchmark.png)](https://h2oai.github.io/db-benchmark/)

Syntax
=======================================================
## Basic

DT[ i, j, by ] 

- i -> on which rows?
- j -> what to do? 
- by -> grouped by what?

## Chaining

DT[i, j, by][i, j, by]...n+1
***

## Special code
- .sd -> Select the columns for the operation
- .() = c() -> List
- .N -> Number of rows in the group
- .GRP -> Number of the group

[Documentation](https://www.rdocumentation.org/packages/data.table/versions/1.12.2/topics/data.table-package)

Join operation
========================================================
type          | operation
------------- | -------------
right join    | DT[X, on="x"]
left join     | X[DT, on="x"]
inner join    | DT[X, on="x", nomatch=NULL]
not join      | DT[!X, on="x"]
join using column "y" of DT with column "v" of X | DT[X, on=c(y="v")
... |


Data Lichess / 20,058 Games
========================================================

```r
library(data.table)
library(dbplyr)
dt <- fread('Data/games.csv')
colnames(dt)
```

```
 [1] "id"             "rated"          "created_at"     "last_move_at"  
 [5] "turns"          "victory_status" "winner"         "increment_code"
 [9] "white_id"       "white_rating"   "black_id"       "black_rating"  
[13] "moves"          "opening_eco"    "opening_name"   "opening_ply"   
```

```
         id rated  created_at last_move_at turns victory_status winner
1: TZJHLljE FALSE 1.50421e+12  1.50421e+12    13      outoftime  white
2: l1NXvwaE  TRUE 1.50413e+12  1.50413e+12    16         resign  black
   increment_code white_id white_rating  black_id black_rating
1:           15+2 bourgris         1500      a-00         1191
2:           5+10     a-00         1322 skinnerua         1261
                                                              moves
1:               d4 d5 c4 c6 cxd5 e6 dxe6 fxe6 Nf3 Bb4+ Nc3 Ba5 Bf4
2: d4 Nc6 e4 e5 f4 f6 dxe5 fxe5 fxe5 Nxe5 Qd4 Nc6 Qe5+ Nxe5 c4 Bb4+
   opening_eco                           opening_name opening_ply
1:         D10       Slav Defense: Exchange Variation           5
2:         B00 Nimzowitsch Defense: Kennedy Variation           4
```


Code example: Select
========================================================
data.tables | dplyr
------------- | -------------
dt[,.('playerWhite' = white_id, 'playerBlack' = black_id, winner)] | df %>% select(playerWhite = white_id, playerBlack = black_id, winner)


```
         playerWhite        playerBlack winner
    1:      bourgris               a-00  white
    2:          a-00          skinnerua  black
    3:        ischia               a-00  white
    4: daniamurashov       adivanov2009  white
    5:     nik221107       adivanov2009  white
   ---                                        
20054:       belcolt           jamboger  white
20055:      jamboger farrukhasomiddinov  black
20056:      jamboger       schaaksmurf3  white
20057:  marcodisogno           jamboger  white
20058:      jamboger              ffbob  black
```


Code example: Where, select, order
========================================================
data.tables | dplyr
------------- | -------------
dt[turns <= 10,.(white_id, black_id, winner, turns)][order(-turns)] | df %>% select(playerWhite = white_id, playerBlack = black_id, winner, turns) %>% filter(turns <= 70) %>% arrange(desc(turns))


```
            white_id          black_id winner turns
    1: hopefullman15           avelez8  black    70
    2:      tohaimr7          isachess  black    70
    3:      maumau62           eideral  black    70
    4:      pablocas           eideral  black    70
    5:     doom12384         vektor-mw  black    70
   ---                                             
13628:    jerusseust         castlebit  white     1
13629: networkchess2         kaskade24  black     1
13630:     lance5500 xxcrunchypebblexx  white     1
13631:    critico-82 lesauteurdeclasse  white     1
13632:    adriansywu      usa-04071776  white     1
```

Code example: Groupby
========================================================
data.tables | dplyr
------------- | -------------
dt_chess <- dt[, .('count' = .N, 'groupNumber' = .GRP), by = opening_name][order(-count)] | df %>% group_by(opening_name) %>% summarise(count = n())  %>% { mutate(ungroup(.), groupNumber = group_indices(.,opening_name)) } %>% arrange(desc(count))


```
                       opening_name count groupNumber
1:             Van't Kruijs Opening   368          11
2:                 Sicilian Defense   358          29
3: Sicilian Defense: Bowdler Attack   296          21
```

Code example: Mean & .SD
========================================================
data.tables | dplyer
------------- | -------------
dt_chess <- dt[, lapply(.SD, mean), .SDcols = columnList]  | df_chess <- df %>% select(columnList) %>% summarise_all(mean)

```
   white_rating black_rating  turns
1:     1596.632     1588.832 60.466
```
Code example: Different operation at once
========================================================
data.tables | dplyer
------------- | -------------
dt[rated == TRUE, list('customMedianWhiteRating' = customMedian(white_rating), 'meanWhiteRating' = mean(white_rating))] | df %>% filter(rated == TRUE)  %>% summarise('customMedianWhiteRating' = customMedian(white_rating), 'meanWhiteRating' = mean(white_rating))


```
   customMedianWhiteRating meanWhiteRating
1:                    1547        1572.363
```

Code example: Add column
========================================================
data.tables | dplyer
------------- | -------------
dt_chess <- dt[, lapply(.SD, mean), by = opening_name, .SDcols = c("white_rating", "black_rating")][order(-white_rating)][,'delta' := white_rating - black_rating] | df_chess <- df %>% group_by(opening_name) %>% summarise(white_rating = mean(white_rating), black_rating = mean(black_rating)) %>% arrange(desc(white_rating)) %>% mutate(delta = white_rating - black_rating)


```
                                                    opening_name
1:               Russian Game: Modern Attack |  Murrey Variation
2:                Queen's Gambit Declined: Westphalian Variation
3:   Tarrasch Defense: Classical Variation |  Carlsbad Variation
4:                        Gruenfeld Defense: Botvinnik Variation
5: English Opening: Anglo-Indian Defense |  Old Indian Formation
   white_rating black_rating delta
1:         2621         1613  1008
2:         2619         1927   692
3:         2586         1618   968
4:         2485         1802   683
5:         2454         1848   606
```

conclusion & Questions
========================================================
- Useful library and its compatible with data.frames and many other libraries.
- Brackets > pipes (personal opinion)
