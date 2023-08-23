---
layout: post
title: "A look at Premier League Wages"
subtitle: "How does each teams wagebill stack up?"
---

### 1. Loading in the Necessary Packages

``` r
library(worldfootballR)
library(tidyverse)
library(reactable)
library(ggthemes)
library(htmltools)
library(scales)
library(ggchicklet)
library(sysfonts)
library(showtext)
```

### 2. Using the WorldFootballR package to read in squad wages for each team and save the data as a csv file

``` r
prem <- fb_league_urls(country = 'ENG', gender = 'M', season_end_year = 2023, tier = "1st")
prem_teams <- fb_teams_urls(prem, time_pause = 3)

wages <- data.frame()

for (i in prem_teams) {
  wage <- fb_squad_wages(team_urls = i)
  wages <- rbind(wages,wage)
}

wages %>%
  arrange(desc(Nation))


#write.csv(wages, 'wages.csv')
wages_csv <- read.csv('wages.csv')
```
### 3. Cleaning the Data and calculating averages for each team

``` r
wages$AnnualWageUSD <- as.integer(wages$AnnualWageUSD)
wages_clean <- wages_csv %>%
  select(Team, 'Name' = Player, 'Nationality' = Nation, 'Position' = Pos, Age, 'Salary' = AnnualWageUSD) %>%
  na.omit() %>%
  mutate(is_english = grepl('ENG', Nationality)) %>%
  arrange(desc(Salary))

wages_clean
wagebyteam <- wages_clean %>%
  group_by(Team) %>%
  summarise(avg = mean(Salary))

wagebyteam <- wagebyteam %>%
  mutate(avg_salary = (avg/1000000)) %>%
  arrange(desc(Team))

wagebyteam$avg_salary <- round(wagebyteam$avg_salary, 3)

#EDITING TEAM NAMES
wagebyteam$Team <- gsub("Wolverhampton Wanderers", "Wolverhampton", wagebyteam$Team)
wagebyteam$Team <- gsub("Tottenham Hotspur", "Tottenham", wagebyteam$Team)
wagebyteam$Team <- gsub("Brighton and Hove Albion", "Brighton", wagebyteam$Team)

wagebyteam$avg_salary <- as.numeric(wagebyteam$avg_salary)
```

### 4. Creating list of team color hex codes and loading in custom fonts from Google Fonts

``` r
teammain <- c('Arsenal' = '#EF0107',
              'Aston Villa' = '#670E36',
              'Bournemouth' = '#DA291C',
              'Brentford' = '#D20000',
              'Brighton' = '#0057B8',
              'Chelsea' = '#034694',
              'Crystal Palace' = '#1B458F',
              'Everton' = '#003399',
              'Fulham' = '#000000',
              'Leeds United' = '#FFCD00',
              'Leicester City' = '#003090',
              'Liverpool' = '#00B2A9',
              'Manchester City' = '#6CABDD',
              'Manchester United' = '#DA291C',
              'Newcastle United' = '#241F20',
              'Nottingham Forest' = '#E53233',
              'Southampton' = '#D71920',
              'Tottenham' = '#132257',
              'West Ham United' = '#7A263A',
              'Wolverhampton' = '#FDB913')

font.add.google(name = 'Source Code Pro', family = 'source-code-pro')
font.add.google(name = 'Staatliches', family = 'staatliches')
showtext.auto()
```

### 5. Plotting the data and customizing the chart using the ggplot2 package

``` r
wagebyteam.epl.plot <- ggplot(data = wagebyteam, aes( y = avg_salary, x = Team, fill = Team)) +
  geom_bar(stat = 'identity', width = .6) +
  geom_chicklet(radius = grid::unit(1.5, "mm")) +
  scale_y_continuous(limits=c(0, 11), breaks=c(0, 2, 4, 6, 8, 10)) +
  scale_fill_manual(values = teammain) +
  coord_flip() +
  theme_wsj() +
  ylab("Average Player Salary (in Millions)") +
  labs(title = "Average Annual Wages") +
  labs(subtitle = "2022-23 Premier League Season") +
  theme(
    plot.title = element_text(size = 22,
                              family = 'staatliches', 
                              color  = '#292727'),
    plot.subtitle = element_text(size=9),
    legend.position = "none",
    axis.title.x  = element_text(
      size = 9, lineheight = 1,
      family = "source-code-pro",
      face = 'plain',
      colour = "#292727"),
    axis.text.y = element_text(
      size = 9, lineheight = 1, 
      family = 'source-code-pro', 
      face = 'plain',
      color = '#292727')
    )
```

![epl_chart](\assets\img\wagebyteam_epl_plot.png)

### 6. Creating an interactive and searchable database of player wages using the reactable package

``` r
wage_search <- wages %>%
  mutate('Percentile' = ntile(WeeklyWageGBP, 100)) %>%
  select(Team, 'Name' =  Player, 'Nationality' = Nation, 'Position' = Pos, WeeklyWageGBP, Percentile) %>%
  na.omit()

wage_search$Team <- gsub("Wolverhampton Wanderers", "Wolverhampton", wage_search$Team)
wage_search$Team <- gsub("Tottenham Hotspur", "Tottenham", wage_search$Team)
wage_search$Team <- gsub("Brighton and Hove Albion", "Brighton", wage_search$Team)

wage_search <- wage_search %>%
  mutate(Nationality = replace(Nationality, Name == 'Alejandro Garnacho', 'ARG')) %>%
  mutate(Nationality = replace(Nationality, Name == 'Samuel Edozie', 'ENG')) %>%
  mutate(Nationality = replace(Nationality, Name == 'Roméo Lavia', 'BEL' )) %>%
  mutate(Nationality = replace(Nationality, Name == 'Juan Larios', 'ESP'))
  
wsj_cream = '#F7F2E6'
highlight_cream = '#FEFCF4'
text_black = '#292727'
glizzy_grey = '#C4C4C4'
green_gradient <- function(x) rgb(colorRamp(c('#F7F2E6','#4DFF6B'))(x), maxColorValue = 255)

wage_search_db <- reactable(wage_search,
          columns = list(
            Team = colDef(width = 200),
            Name = colDef(width = 250),
            Nationality = colDef(name = 'NAT',
                                 align = 'center', 
                                 width = 75),
            Position = colDef(name = 'POS',
                              align = 'center',
                              width = 75,),
            WeeklyWageGBP = colDef(name = "Weekly Wage (£)", 
                                   align = 'center',
                                   width = 200,
                                   format = colFormat(separators = TRUE)),
            Percentile = colDef(name = '%', 
                                align = 'center', 
                                width = 75, 
                                style = function(value) {
                                  normalized <- (value - min(wage_search$Percentile)) /   (max(wage_search$Percentile) - min(wage_search$Percentile))
                                  color <- green_gradient(normalized)
                                  list(background = color)
                                })),
          language = reactableLang(searchPlaceholder = 'Filter Data',
                                   noData = 'No Results'),
          theme = reactableTheme(
            backgroundColor = wsj_cream,
            highlightColor = highlight_cream,
            borderColor = glizzy_grey,
            cellPadding = '8px 12px',
            style = list(fontFamily = 'Roboto Mono',
                         fontSize = '15px',
                         color = text_black),
            searchInputStyle = list(width = '100%',
                                    backgroundColor = wsj_cream,
                                    color = text_black,
                                    borderColor = glizzy_grey)),
                                    
          defaultSorted = 'Name',
          fullWidth = FALSE,
          searchable = TRUE, 
          highlight = TRUE,
          bordered = TRUE,
          wrap = FALSE,
          paginationType = "simple",
          minRows = 10)

```
<iframe src="\assets\img\wage.search.db.html" height='600px' width='100%'></iframe>
