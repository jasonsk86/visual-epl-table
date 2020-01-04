---
title: "Visual Premier League Table"
author: "Jason Skelton"
date: "04/01/2020"
output: 
  html_document:
    toc: true
    keep_md: true
---




## Intro

The goal is to create a visual representation of the Premier League Table that better shows the distance between teams based on points differential.

The dataset I'll be working with is the premier league data scraped from <https://www.skysports.com/premier-league-table>.


## Scraping the data with Python

First I'll need to load the **reticulate** package to work with Python in R, and point to the version of python I want to use.


```r
library(reticulate)
use_python('C:/Users/skeltonj/AppData/Local/Continuum/python')
```

I'll use the BeautifulSoup package for web scraping.


```python
from bs4 import BeautifulSoup
import requests
```



```python
url = 'https://www.skysports.com/premier-league-table'

page = requests.get(url)
html = page.text
html = BeautifulSoup(html, 'html.parser')
```

I get the league data by iterating through all the neccesary elements and storing them as a list. I just used the Chrome developer tools to identify which tags and classes I needed to scrape for the data.


```python
headers = [x.text.strip() for x in html.find_all('th', class_ = 'standing-table__header-cell')]
league_data = [x.text.strip() for x in html.find_all('td', class_ = 'standing-table__cell')]

league_data
```

```
## ['1', 'Liverpool', '20', '19', '1', '0', '49', '14', '35', '58', '', '2', 'Leicester City', '21', '14', '3', '4', '46', '19', '27', '45', '', '3', 'Manchester City', '21', '14', '2', '5', '56', '24', '32', '44', '', '4', 'Chelsea', '21', '11', '3', '7', '36', '29', '7', '36', '', '5', 'Manchester United', '21', '8', '7', '6', '32', '25', '7', '31', '', '6', 'Tottenham Hotspur', '21', '8', '6', '7', '36', '30', '6', '30', '', '7', 'Wolverhampton Wanderers', '21', '7', '9', '5', '30', '27', '3', '30', '', '8', 'Sheffield United', '21', '7', '8', '6', '23', '21', '2', '29', '', '9', 'Crystal Palace', '21', '7', '7', '7', '19', '23', '-4', '28', '', '10', 'Arsenal', '21', '6', '9', '6', '28', '30', '-2', '27', '', '11', 'Everton', '21', '7', '4', '10', '24', '32', '-8', '25', '', '12', 'Southampton', '21', '7', '4', '10', '25', '38', '-13', '25', '', '13', 'Newcastle United', '21', '7', '4', '10', '20', '33', '-13', '25', '', '14', 'Brighton and Hove Albion', '21', '6', '6', '9', '25', '29', '-4', '24', '', '15', 'Burnley', '21', '7', '3', '11', '24', '34', '-10', '24', '', '16', 'West Ham United', '20', '6', '4', '10', '25', '32', '-7', '22', '', '17', 'Aston Villa', '21', '6', '3', '12', '27', '37', '-10', '21', '', '18', 'Bournemouth', '21', '5', '5', '11', '20', '32', '-12', '20', '', '19', 'Watford', '21', '4', '7', '10', '17', '34', '-17', '19', '', '20', 'Norwich City', '21', '3', '5', '13', '22', '41', '-19', '14', '']
```

I could rework this data into the right format in Python using NumPy and Pandas, however I'm more comfortable in R so instead I'll just pass these python lists over to R and complete the job from there.

## Shaping the data with R

Now I'll work with the scraped data from Python and manipulate into the correct format using R.


```r
library(tidyverse)
library(ggrepel)
library(ggimage)
```

Here I pass the lists from python to R and store them as character vectors. I then re-shape into a matrix and convert to a data frame.


```r
cols <- py$headers
data <- py$league_data

table <- matrix(data, nrow = 20, byrow = TRUE) 
table <- as.tibble(table)
names(table) <- cols

table$`Last 6` <- NULL

table
```

```
## # A tibble: 20 x 10
##    `#`   Team               Pl    W     D     L     F     A     GD    Pts  
##    <chr> <chr>              <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
##  1 1     Liverpool          20    19    1     0     49    14    35    58   
##  2 2     Leicester City     21    14    3     4     46    19    27    45   
##  3 3     Manchester City    21    14    2     5     56    24    32    44   
##  4 4     Chelsea            21    11    3     7     36    29    7     36   
##  5 5     Manchester United  21    8     7     6     32    25    7     31   
##  6 6     Tottenham Hotspur  21    8     6     7     36    30    6     30   
##  7 7     Wolverhampton Wan~ 21    7     9     5     30    27    3     30   
##  8 8     Sheffield United   21    7     8     6     23    21    2     29   
##  9 9     Crystal Palace     21    7     7     7     19    23    -4    28   
## 10 10    Arsenal            21    6     9     6     28    30    -2    27   
## 11 11    Everton            21    7     4     10    24    32    -8    25   
## 12 12    Southampton        21    7     4     10    25    38    -13   25   
## 13 13    Newcastle United   21    7     4     10    20    33    -13   25   
## 14 14    Brighton and Hove~ 21    6     6     9     25    29    -4    24   
## 15 15    Burnley            21    7     3     11    24    34    -10   24   
## 16 16    West Ham United    20    6     4     10    25    32    -7    22   
## 17 17    Aston Villa        21    6     3     12    27    37    -10   21   
## 18 18    Bournemouth        21    5     5     11    20    32    -12   20   
## 19 19    Watford            21    4     7     10    17    34    -17   19   
## 20 20    Norwich City       21    3     5     13    22    41    -19   14
```

Since we created this from lists of charcters from Python, we'll need to convert numeric data properly.


```r
table <- table %>% 
  mutate_at(vars(Pl:Pts), ~as.numeric(as.character(.)))
```


## Visualizing the table with ggplot

Lastly, now that I have all the data in the right shape, I'll just use **ggplot** to viusalise it. 

To help separate out teams that are on the same points tally, I'll use **row_number()** and then use that to plot the teams across the x-axis so that teams on the same points don't overlap.


```r
table <- table %>% 
  group_by(Pts) %>% 
  mutate(shared = row_number()) %>% 
  ungroup()
```

I'm adding a club colour as a hex value to the table.


```r
cols <- list(Arsenal = '#fe0002', AstonVilla = '#480025', Bournemouth = '#e62333', BrightonandHoveAlbion = '#0054a6',
             Burnley = '#6a003a', Chelsea = '#0a4595', CrystalPalace = '#eb302e', Everton = '#00369c',
             LeicesterCity = '#273e8a', Liverpool = '#e31b23', ManchesterCity = '#6caee0', 
             ManchesterUnited = '#d81920', NewcastleUnited = '#383838', NorwichCity = '#00a94f', 
             SheffieldUnited = '#ed1c24', Southampton = '#d71920', TottenhamHotspur = '#f5f5f5', Watford = '#ffee00',
             WestHamUnited = '#7d2c3b', WolverhamptonWanderers = '#f9a01b')

club_col <- as.tibble(cols) %>% 
  pivot_longer(
    cols = everything(),
    names_to = "Team_clean",
    values_to = "club_color"
  )


table$Team_clean <- str_replace_all(table$Team, "[[:space:]]", "")

table <- table %>% 
  left_join(club_col, by= "Team_clean")
```

Here I'm setting up a mapping of club colours to the team to apply to the plot. 


```r
clubCol <- table$club_color
names(clubCol) <- table$Team
head(clubCol)
```

```
##         Liverpool    Leicester City   Manchester City           Chelsea 
##         "#e31b23"         "#273e8a"         "#6caee0"         "#0a4595" 
## Manchester United Tottenham Hotspur 
##         "#d81920"         "#f5f5f5"
```

Finally, the visual table. 


```r
ggplot(table, aes(x = shared, y = Pts, label = Team, fill = Team)) +
  geom_tile(color = "white", size = 1) +
  geom_text(aes(label = Team), color = "white") +
  ggthemes::theme_fivethirtyeight() +
  scale_fill_manual(values = clubCol) +
  scale_y_continuous(breaks = seq(0, max(table$Pts), 3)) +
  labs(title = "A Visual Representation of the Premier League Table",
       y = "Points",
       x = "",
       caption = "Data from: SkySports  |  Created by: Jason Skelton") +
  theme(
    legend.position = "none",
    axis.text.x = element_blank(),
    axis.title = element_text(),
    panel.grid.major = element_blank(),
    axis.line = element_line(color = "darkgray"),
    axis.ticks = element_line(color = "darkgray")
  ) 
```

![](visual_premier_league_table_files/figure-html/visualize-table-1.png)<!-- -->



