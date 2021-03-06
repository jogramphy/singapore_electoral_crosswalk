# Background and Motivation

The 2020 General Election in Singapore was held on July 10 2020, with the ruling People's Action Party (PAP) consolidating its power by winning 83 out of 93 seats available. However, while this seemed like a dominant victory (which it is), there were some close fights; in particular the PAP edged out the opposition at West Coast GRC (Group Representative Constitutency), winning 51.69% of the votes (Rajan et al., 2020) <sup>1<sup>


The 2020 elections were not like other elections. Notwithstanding the unusual circumstances of holding an election in the middle of a pandemic, there were other controversies too, like the withdrawal of a PAP member due to allegations of "elitism and arrogant behaviour at work" (Sim & Jaipragas, 2020) <sup>2<sup> 

This led to more eyeballs than usual on this election, with citizens asking important questions: 
  + How did different electoral boundaries vote? 
  + Are there socio-economic factors (like income etc.) that affect these voting patterns? 

The first question is easy to answer - the Singaporean government publishes electoral data very publically. However, the second is much harder to answer. Singapore does not publicize or share data categorized by electoral boundaries. Unlike American census data where one can aggregate tract level data to state level and visualize on a map election patterns across the country, Singapore does not have a similar data synthesis process. By virtue of being a city-state, the country is split into ‘planning areas’ by the Urban Redevelopment Authority (URA). Census information is collected at the planning area level, but these planning areas are not congruent with the electoral boundaries set in place by the Elections Department (ELD). This leads to our inability to map out differences in indicators across electoral boundaries. 

# Goals and Objectives

I am interested in examining the spatial distribution of income and education within Singapore’s electoral boundaries, specifically in the 2020 General Elections. The final product will be an R Shiny App and statistical visualizations. Specifically, the dashboard will allow users to:
- View data tables about their electoral boundaries
- Interact with a map that shows the distribution of income and education level across electoral boundaries 

Underlying the above goals and objectives, fundamental to the project is to develop a framework to transpose the electoral boundaries to the boundaries of the Singapore Planning Areas. Developing such a framework will allow us to examine changes of electoral boundaries, the associated indicators across more dimensions such as time. The output for this will be a `crosswalk.csv` file that will allow future researchers to utilize. 

# Data Sources

| Dataset Name      | File Type | Source      | 
| ----------- | ----------- | ----------- |
| Electoral Boundary 2020 <sup>a<sup>     | KMZ       | https://data.gov.sg/dataset/electoral-boundary_2020       |
| Master Plan 2014 Planning Area Boundary <sup>b<sup>  | SHP        | https://data.gov.sg/dataset/master-plan-2014-planning-area-boundary-web?resource_id=2ab23cb2-b1a4-4b1a-a9e1-b9cad0ac159b |
| Resident Population Aged 15 Years and Over by Planning Area and Highest Qualification Attained, 2015 <sup>c<sup> | CSV       | https://data.gov.sg/dataset/resident-population-aged-15-years-and-over-by-planning-area-and-highest-qualification-attained-2015 |
| Resident Working Persons Aged 15 Years and Over by Planning Area and Gross Monthly Income from Work, 2015 <sup>d<sup>| CSV       | https://data.gov.sg/dataset/resident-working-persons-aged-15-years-over-by-planning-area-gross-monthly-income-from-work-2015 |
| Parliamentary General Election Results, 2020 <sup>e<sup>| CSV       | https://data.gov.sg/dataset/parliamentary-general-election-results |

We use the 2015 Planning Area data, comparing it against results from the 2020 General Elections. More specifically, the data in 2015 is taken from the General Household Survey, which is conducted between 2 population censuses as a mid-decade census. We do not use 2020 data because it is not available yet, and that citizens in 2020 vote based on the reality of the past few years, and not on the 'latest available data', strictly speaking. 


# Methods Used 

## Extract, Transform, Load 

#### Loading the Shapefiles into our Environment
We first want to extract and load the relevant datasets to our environment. Some of our files are in SHP, and some of them are in KMZ - we intend to standardize everything to SHP files. Due to the nature of our `gdal` installation, we are unable to read .kmz files. We utilize https://mygeodata.cloud (an open source platform) to help us convert the data from kmz to shp for easier crosswalking calculations. 

```{r }
eld <- st_read("data/elections/eld.shp") # Electoral Boundaries File
planning <- st_read("data/planning/MP14_PLNG_AREA_WEB_PL.shp") 
```
#### Visualizing the Shapefiles
We map out the initial shapefiles to see what we are working with. We use the `tmap` package to help us do this. First, the Election Boundaries map: 

![image](/visualizations/singapore_ge_2020.png)

Secondly, the Planning Areas map: 

![image](/visualizations/singapore_planning_area_2015.png)

More importantly, what we want to do is to overlay one file on the other and see how it inersects. We follow the example outlined to help us do this, as outlined by Tannen (2021) <sup>3<sup> 

We first include some formatting instructions so that we can do the overlaying seamlessly. 

```{r}
sixtysix_colors <- list(
  red="#D0445E",
  blue="#0077E0",
  purple="#C92EC4",
  green="#009871",
  orange="#9C7200",
  grey="#009871",
  light_red="#EE6178"
)

theme_map_sixtysix <- function(){ 
  ggthemes::theme_map() %+replace%
    theme(
      text = ggthemes::theme_fivethirtyeight()$text,
      title = ggthemes::theme_fivethirtyeight()$plot.title,
      panel.grid.major = element_line(color="grey90")
    )
}

theme_map_sixtysix <- function(){ 
  ggthemes::theme_map() %+replace%
    theme(
      text = ggthemes::theme_fivethirtyeight()$text,
      title = ggthemes::theme_fivethirtyeight()$plot.title,
      panel.grid.major = element_line(color="grey90")
    )
}
```
Then, we map the 2 maps, one over the other. 

```{r}
#Creatihng overlapping maps

ggplot(
  eld
  ) +
    geom_sf(
    color = sixtysix_colors$light_red,
    size=1, 
    fill=NA
  ) + 
  geom_text(
    aes(x=X, y=Y, label = ED_CODE),
    data = with(
      eld,
      data.frame(ED_CODE, st_centroid(geometry) %>% st_coordinates())
    ),
    color=sixtysix_colors$light_red,
    size=4,
    fontface="bold"
  ) +
  geom_sf(
    data=planning,
    color = "black",
    size=1,
    fill=NA
  ) +
  geom_text(
    aes(x=X, y=Y, label = PLN_AREA_C),
    data = with(
      planning,
      data.frame(PLN_AREA_C, st_centroid(geometry) %>% st_coordinates())
    ),
    color="black",
    size=4
  ) +
  scale_color_identity(guide=FALSE) +
  theme_map_sixtysix() +
  ggtitle("Election Boundaries (Red) vs \n Planning Areas (Black)")
```

The resulting map: 

![image](/visualizations/election_planning_overlaid.png)


## Crosswalk Formulation 
A crosswalk is required for us to map data across "different levels of geographical aggregation" (Eckert et al., 2020) <sup>4<sup> 

The crosswalk is essentially a list of weights that allows us to transpose data from one geographical aggregation to another. There are two approaches that we can adopt: Areal Weighting, and Population Weighting. 

**Areal Weighting** involves proportioning values by land area. For example, if 50% of the Jurong electoral boundary is in Planning Area A, we give 50% of Planning Area A's values to Jurong, and regroup all of Jurong's value by sum subsequently.  **Population weighting** involves proportioning values by population of people. We will utilize areal weighting for this project given and considering its context. Singapore as a country is that of high population density - and there really is not an explicit "rural" or "urban" divide that will make population weighting more useful than an areal one. 

```{r}
areal_weights <- st_intersection(
  st_make_valid(planning), 
  st_make_valid(eld)
  ) %>%
  mutate(area = st_area(geometry)) %>%
  as.data.frame() %>%
  select(PLN_AREA_N, Name, area) %>% 
  group_by(PLN_AREA_N) %>%
  mutate(prop_of_pln = as.numeric(area / sum(area))
  )
```

Let us do some exploration first and see what the crosswalk "means". 
```{r}
areal_weights %>% filter(Name == "HOLLAND-BUKIT TIMAH")
``` 

| pln_area     | eld_bounds| area      | prop_of_pln |
| ----------- | ----------- | ----------- | ----------- | 
| BUKIT BATOK    | HOLLAND-BUKIT TIMAH        | 284209.813   | 0.025528053 |
| BUKIT PANJANG    | HOLLAND-BUKIT TIMAH        | 7312727.286    | 0.810729970 |
| BUKIT TIMAH    | HOLLAND-BUKIT TIMAH        | 13358327.473  | 0.762172615  |

Here we see a snippet of the crosswalk. We see that to obtain data for the Holland-Bukit Timah constitutency (the electoral boundary I personally belong to), I would take 0.02552 of Bukit Batok's data, 0.81 of Bukit Panjang's, etc. We note that there are also some values in `prop_of_pln` that are very close to zero. These are areas that are probably very trivial (i.e. a very very small intersection) but we will still count them in our final data output for the sake of completeness. 

```{r}
crosswalk = 
  areal_weights %>%
  rename(eld_bounds = Name) %>%
  rename(pln_area = PLN_AREA_N)

write.csv(crosswalk, file="crosswalk.csv")
```

## Using the Crosswalk
To use the crosswalk, we first have to load in a relevant dataset. We will work with 2 datasets; one that deals with gross income levels in 2015, and the other concerning the highest education qualification attained. Both are at the planning area level - we want to transpose them to the electoral planning level. 

### Input and Clean Income Dataset
```{r}
income_pln <- read_csv("data/raw/income.csv")
# Let's just keep the columns that we want and throw out the rest.

clean_inc =
income_pln %>%
  select(level_1, level_3, value) %>% 
  rename(breakdown = level_1) %>%
  rename(pln_area = level_3)

joined_income = 
  left_join(clean_inc, crosswalk, by = "pln_area") %>% 
  mutate(scaled = as.numeric(value * prop_of_pln))

```
Using the joined dataset, we group by breakdown (which is the "income level") and electoral bounds. We then sum up the "scaled" values per intersection (e.g. Planning Area 1 intersects with Election Bound A, Planning Area 2 intersects with Election Bound A), and then group by electoral bounds.
```{r}
test_income = 
  joined_income %>% 
  group_by(breakdown, eld_bounds) %>%
  mutate(sum_eld = sum(scaled)) %>%
  arrange(breakdown, eld_bounds)
```

To better understand this, let's just filter out a small proportion of the dataset to see what is going on.

```{r}
test_income %>% filter(eld_bounds == "HOLLAND-BUKIT TIMAH", breakdown == "$2,000 - $2,999")
```

This dataset tells us that within the Holland-Bukit Timah Constituency, the 'value' of those earning between $2,000 - $2,999 of income is 13.60. Values in this case represent the number of people [Resident Working Persons Aged 15 Years and Over by Planning Area and Gross Monthly Income from Work, 2015], in thousands. What we want to do is to convert each value to a percentage of that election bounds's population. 

First, we need to calculate the electoral bounds's total population (our denominator, if you will). 
```{r}
test_income %>% 
  filter(breakdown == "Total")
```

To calculate Aljunied's total population, we will 'search up' Aljunied in `eld_bounds`, sum up the corresponding values in `sum_eld`, and then divide it by the number of rows in Aljunied. 
```{r}
total_income =
test_income %>% 
  filter(breakdown == "Total") %>%
  group_by(eld_bounds) %>% 
  summarise(pop_in_000 = mean(sum_eld)) ## Mean is the same as adding and then dividing!
  ```

We can write a function that extracts the correct population value for that level. 

```{r}
## List of Levels
inc_levels <- list('Below $1,000', 
                   '$1,000 - $1,999',
                   '$2,000 - $2,999', 
                   '$3,000 - $3,999', 
                   '$4,000 - $4,999',
                   '$5,000 - $5,999', 
                   '$6,000 - $6,999', 
                   '$7,000 - $7,999', 
                   '$8,000 - $8,999',
                   '$9,000 - $9,999', 
                   '$10,000 - $10,999', 
                   '$11,000 - $11,999', 
                   '$12,000 & Over'
                   )

#Calculate each level of income as a percentage of total population
income_value_pct <- function(level) {
  test_income %>%
    filter(breakdown == level) %>%
    group_by(eld_bounds) %>% 
    summarise(pop_in_000 = mean(sum_eld)) %>% 
    mutate(pct = 100* (pop_in_000 / total_income$pop_in_000)) %>%
    mutate(inc_level = level) # We include this so we know what we are looking at 
}

eld_income <- lapply(inc_levels, income_value_pct) %>% bind_rows() %>% drop_na()
```

### Input and Clean Education Dataset
We do the same with the education dataset. 
```{r}
edu_pln <- read.csv("data/raw/edu_pln.csv")

# Let's just keep the columns that we want and throw out the rest.
clean_edu =
edu_pln %>%
  select(level_1, level_3, value) %>% 
  rename(breakdown = level_1) %>%
  rename(pln_area = level_3)

joined_edu = 
  left_join(clean_edu, crosswalk, by = "pln_area") %>% 
  mutate(scaled = as.numeric(value * prop_of_pln))

test_edu = 
  joined_edu %>% 
  group_by(breakdown, eld_bounds) %>%
  mutate(sum_eld = sum(scaled)) %>%
  arrange(breakdown, eld_bounds)

```

Values in this case represent the number of people in thousands and their respective highest education attained. We convert this to a percentage value of total population.  First, we need to calculate the electoral bounds's total population (our denominator, if you will). 

```{r}
total_edu =
test_edu %>% 
  filter(breakdown == "Total") %>%
  group_by(eld_bounds) %>% 
  summarise(pop_in_000 = mean(sum_eld)) ## Mean is the same as adding and then dividing!

```

We can write a function that extracts the correct population value for that level. 

```{r}
## List of Levels
edu_levels <- list('No Qualification', 
                   'Primary',
                   'Lower Secondary', 
                   'Secondary', 
                   'Post-Secondary (Non-Tertiary)',
                   'Polytechnic', 
                   'Professional Qualification and Other Diploma', 
                   'University'
                   )

#Calculate each level of edu as a percentage of total population
edu_value_pct <- function(level) {
  test_edu %>%
    filter(breakdown == level) %>%
    group_by(eld_bounds) %>% 
    summarise(pop_in_000 = mean(sum_eld)) %>% 
    mutate(pct = 100* (pop_in_000 / total_edu$pop_in_000)) %>%
    mutate(edu_level = level) # We include this so we know what we are looking at 
}

eld_edu <- lapply(edu_levels, edu_value_pct) %>% bind_rows() %>% drop_na()
```
### Input and Clean Voting Data

We also import voting data. We express voting data in the column `vote_pap_pct`, which shows the percentage of the population that voted for the People's Action Party, who is the ruling party that contests in every single constitutency (electoral area). 
```{r}
voting <- read_csv("data/raw/voting.csv")
voted_pap = voting %>% filter(year==2020, party=="PAP") %>% mutate(vote_pap_pct = as.numeric(vote_percentage) * 100) %>% select(constituency, vote_pap_pct) # Data from the 2020 Elections
```

We then join numeric data to the main dataset of electoral data. 

```{r}

# We pivot the data to get the income and edu data
pivoted_income = eld_income %>% select(-"pop_in_000") %>% pivot_wider(names_from = inc_level, values_from = pct) 
pivoted_edu = eld_edu %>% select(-"pop_in_000") %>% pivot_wider(names_from = edu_level, values_from = pct)

eld_2 = eld %>% select(c("Name", "geometry")) # We make a copy of the Electoral Boundary file with the columns we want

join_income_eld = inner_join(eld_2, pivoted_income, by=c("Name"='eld_bounds')) # We join income data to eld data first
join_edu_data = inner_join(join_income_eld, pivoted_edu, by=c("Name"="eld_bounds")) # Join Edu Data with income Data
eld_data = 
  inner_join(join_edu_data, voted_pap, by=c("Name"="constituency")) %>%
  mutate(across(where(is.numeric), round, 2)) #Round values
```

Now that we have a completed and cleaned datasaet at the electoral boundary level, we can begin to create some cartographical visuatlizations.

## Visualization

### Visualizing Income Data

We want to generate a bar chart that allows us to compare percentages of different income levels across boundaries, as shown.

```{r, fig.width=15, fig.height=15}

ggplot(eld_income, aes(fill=factor(inc_level, levels=rev(inc_levels)), y=pct, x=eld_bounds)) + 
  geom_bar(position="fill", stat="identity") + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1, size=20),
        axis.title.x = element_text(size=30),
        axis.title.y = element_text(size = 30),
        axis.text.y = element_text(size=20),
        legend.box.background = element_rect(),
        legend.box.margin = margin(6, 6, 6, 6), 
        legend.key.size = unit(2,'cm'), 
        legend.title = element_text(size=18, face='bold'),
        legend.text = element_text(size=15)) + xlab("Electoral Boundary Name") +
  ylab("Percentage of Population") + labs(fill = "Income Levels") + coord_flip()
```

The image of the bar chart will be shown in the results section below. 

### Visualizing Education Data

The same process works for the education data: 
```{r, fig.width=15, fig.height=15}
#Make education barplot
ggplot(eld_edu, aes(fill=factor(edu_level, levels=rev(edu_levels)), y=pct, x=eld_bounds)) + 
  geom_bar(position="fill", stat="identity") + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1, size=20),
        axis.title.x = element_text(size=30),
        axis.title.y = element_text(size = 30),
        axis.text.y = element_text(size=20),
        legend.box.background = element_rect(),
        legend.box.margin = margin(6, 6, 6, 6), 
        legend.key.size = unit(2,'cm'), 
        legend.title = element_text(size=18, face='bold'),
        legend.text = element_text(size=15)) + xlab("Electoral Boundary Name") +
  ylab("Percentage of Population") + labs(fill = "Education Levels") + coord_flip()

```

The image of the bar chart will be shown in the results section below. 

### Combining Income and Education Data with Voting Data 

We also want to generate some cartographical visualizations. First, we generate maps that show the percentage of the population that voted for the ruling People's Action Party. Before we do that, let us make some custom color palettes to visualize our data. 

```{r}
mypal <- c('#edf8fb', '#b3cde3', '#8c96c6','#8856a7','#810f7c')
tmap_mode("plot")
tm_shape(eld_data)  + tm_polygons("vote_pap_pct", title = "Percentage Voted for PAP in 2020 Elections", palette = mypal) + tm_borders() +  tm_text("Name", size = 1/3)
tmap_mode("view")
income_map = 
  tm_shape(eld_data) + tm_text("Name", size = 1/1.3) + tm_borders(lwd = 2) + 
  tm_polygons("vote_pap_pct", title = "Percentage Voted for PAP",
              popup.vars=c(
                "Income Below $1,000: " = "Below $1,000", 
                "Income $1,000 - $1,999: " = "$1,000 - $1,999",
                "Income $2,000 - $2,999: " = "$2,000 - $2,999",
                "Income $3,000 - $3,999: " = "$3,000 - $3,999",
                "Income $4,000 - $4,999: " = "$4,000 - $4,999",
                "Income $5,000 - $5,999: " = "$5,000 - $5,999",
                "Income $6,000 - $6,999: " = "$6,000 - $6,999",
                "Income $7,000 - $7,999: " = "$7,000 - $7,999",
                "Income $8,000 - $8,999: " = "$8,000 - $8,999",
                "Income $9,000 - $9,999: " = "$9,000 - $9,999",
                "Income $10,000 - $10,999: " = "$10,000 - $10,999",
                "Income $11,000 - $11,999: " = "$11,000 - $11,999",
                "Income above $12,000: " = "$12,000 & Over", 
                "Percentage who voted for PAP: " = "vote_pap_pct")
                ) + tm_layout(title="Income (2015) and PAP Vote Share (2020)")

tmap_leaflet(income_map, show = TRUE, add.titles=TRUE)
```
This gives us this income map: 
![Income Leaflet](/visualizations/income_map_sample.png)

For education, the same code with slight modifications: 
```{r}
edu_map = 
  tm_shape(eld_data) + tm_text("Name", size = 1/1.3) + tm_borders(lwd = 2) + 
  tm_polygons("vote_pap_pct", title = "Percentage Voted for PAP",
              popup.vars=c(
                "No Qualification: " = "No Qualification", 
                "Primary: " = "Primary", 
                "Lower Secondary: " ="Secondary", 
                "Secondary: " = "Secondary",
                "Post-Secondary (Non-Tertiary): " = "Post-Secondary (Non-Tertiary)",
                "Polytechnic: " = "Polytechnic",
                "Professional Qualification and Other Diploma: " = "Professional Qualification and Other Diploma",
                "University: " = "University", 
                "Percentage who voted for PAP: " = "vote_pap_pct")
                ) + tm_layout(title="Highest Edu Qualification (2015) and PAP Vote Share (2020)")

tmap_leaflet(edu_map, show = TRUE, add.titles=TRUE)
```
This gives us this education map:
![Education Leaflet](/visualizations/education_map_sample.png)

## R Shiny Development 

We take the visualizations a step further by creating an R Shiny app that allows us to view the data in a centralized fashion. The R Shiny App should allow us to compare data in an interactive manner at the electoral boundary level between electoral boundaries. The file can be found in `shiny_gis3.R`, but it is embedded inline in the `crosswalk-r.rmd` file. The code is replicated below: 

```{r, echo=FALSE}
shinyApp(

  ui = fluidPage(
  
  # App title ----
  titlePanel("Singapore Electoral Boundary Data Explorer"),
  
  # Sidebar layout with input and output definitions ----
  sidebarLayout(
    sidebarPanel(
      h3("Make your choice"),
      helpText("Explore Singapore data based on selected variables"),
      selectInput(
      "selectedCol1",
      "Select 1st boundary to compare",
      choices = list("ALJUNIED",
                     "ANG MO KIO",
                     "BISHAN-TOA PAYOH",
                     "BUKIT BATOK",
                     "BUKIT PANJANG",
                     "CHUA CHU KANG",
                     "EAST COAST",
                     "HOLLAND-BUKIT TIMAH",
                     "HONG KAH NORTH",
                     "HOUGANG",
                     "JALAN BESAR",
                     "JURONG",
                     "KEBUN BARU",
                     "MACPHERSON",
                     "MARINE PARADE",
                     "MARSILING-YEW TEE",
                     "MARYMOUNT",
                     "MOUNTBATTEN",
                     "NEE SOON",
                     "PASIR RIS-PUNGGOL",
                     "PIONEER",
                     "POTONG PASIR",
                     "PUNGGOL WEST",
                     "RADIN MAS",
                     "SEMBAWANG",
                     "SENGKANG",
                     "TAMPINES",
                     "TANJONG PAGAR",
                     "WEST COAST",
                     "YIO CHU KANG",
                     "YUHUA"),
      selected = "Bound 1"
    ),
    selectInput(
      "selectedCol2",
      "Select 2st boundary to compare",
      choices = list("ALJUNIED",
                     "ANG MO KIO",
                     "BISHAN-TOA PAYOH",
                     "BUKIT BATOK",
                     "BUKIT PANJANG",
                     "CHUA CHU KANG",
                     "EAST COAST",
                     "HOLLAND-BUKIT TIMAH",
                     "HONG KAH NORTH",
                     "HOUGANG",
                     "JALAN BESAR",
                     "JURONG",
                     "KEBUN BARU",
                     "MACPHERSON",
                     "MARINE PARADE",
                     "MARSILING-YEW TEE",
                     "MARYMOUNT",
                     "MOUNTBATTEN",
                     "NEE SOON",
                     "PASIR RIS-PUNGGOL",
                     "PIONEER",
                     "POTONG PASIR",
                     "PUNGGOL WEST",
                     "RADIN MAS",
                     "SEMBAWANG",
                     "SENGKANG",
                     "TAMPINES",
                     "TANJONG PAGAR",
                     "WEST COAST",
                     "YIO CHU KANG",
                     "YUHUA"),
      selected = "Bound 2"
    ),
    selectInput("Map_Choice", 
                label = "View electoral boundary level data on an interactive map of Singapore",
                choices = list("Income Data", 
                               "Education Data"),
                selected = "Income Data"),
    
    h4("About"),
    p("This project was completed by Josea Evan, as part of coursework for GIS III at the University of Chicago. Contact: josea [at] uchicago.edu"),
    h4("Data Source"),
    p("This data is from the Singapore Census. Electoral Data from 2020 Singapore General Election. Income and Education Data crosswalked from 2015 Household Survey.")
  ),
    
    # Main panel for displaying outputs ----
    mainPanel(
      # Output: Tabset w/ plot, summary, and table ----
      tabsetPanel(type = "tabs",
                  tabPanel("Education Data", plotOutput("distPlot")),
                  tabPanel("Income Data", plotOutput("incPlot")),
                  tabPanel("Map", leafletOutput("working_map"))

      )
      
    )
  )
),

  server <- function(input, output) {
  
  output$distPlot <- renderPlot({
    test_edu = 
      eld_edu %>% filter(eld_bounds %in% c(input$selectedCol1, input$selectedCol2))
    
    
    ggplot(test_edu, aes(fill=factor(edu_level, levels=rev(edu_levels)), y=pct, x= "Boundary Name")) + 
      geom_bar(position="fill", stat="identity") + facet_grid(~eld_bounds) + 
      theme(legend.box.background = element_rect(),
            legend.title = element_text(face='bold')) + 
      labs(fill = "Highest Education Level Attained (2015)") +
      ylab("Percentage of Population") 
    
  })

  #Generate a Violin Plot of Variables 
  output$incPlot <- renderPlot({
    test_inc = 
      eld_income %>% filter(eld_bounds %in% c(input$selectedCol1, input$selectedCol2))
    
    
    ggplot(test_inc, aes(fill=factor(inc_level, levels=rev(inc_levels)), y=pct, x= "Boundary Name")) + 
      geom_bar(position="fill", stat="identity") + facet_grid(~eld_bounds) + 
      theme(legend.box.background = element_rect(),
            legend.title = element_text(face='bold')) + 
      labs(fill = "Gross Monthly Income from Work (2015)") +
      ylab("Percentage of Population") 
  })
  
  output$working_map <- renderLeaflet({
    
    map <- switch(input$Map_Choice, 
                  "Income Data" = income_map,
                  "Education Data" = edu_map)
    
    working_map <- tmap_leaflet(map)
    
  })
},

  options = list(height = 800)
)
```

# Results

## Bar Chart Results 

We initially plotted bar charts that allowed comparison of income levels across all electoral boundaries. For income, we see the bar chart comparison of income in this figure:
![Income_bar_chart](/visualizations/income_charts.png) 

For education levels, we see this stacked bar chart as shown below: 
![Edu-bar-chart](/visualizations/edu_charts.png)

## R Shiny Sample 

This is how the R Shiny App looks like. You see on the left sidebar, there are drop down menus that allow you to select which variables to compare. Clicking on the dropdown generates a view such as this: 
![Drop Down](/visualizations/drop_down.png)

Selecting two different boundaries will generate a side by side bar chart that allows you to compare education, and income data by percentage on the electoral boundary level. 

There are tabs on the main panel that you can click through. 

The first tab lets you compare education levels:
![Education Comparison](/visualizations/edu_bar_sample.png)

The second tab lets you compare income:
![Income Comparison](/visualizations/income_bar_sample.png)

The third tab lets you compare the maps of income and education data (you select income or education from the side panel). The generated map is interactive, with the popup that highlights income/education level against percentage voted for the People's Action Party. 

Income Map:
![Income Map](/visualizations/income_sample.png)

Education Map:
![Education Map](/visualizations/edu_sample.png)

# Main Highlights

The results of this project are three fold. First, we have successfully produced a crosswalk file, in the form of the `crosswalk.csv` file, avaiable in this respository. The csv file allows other users to be able to project planning area data (which is how most data is stored in the data.gov website) to electoral boundary level data. This will help us understand variations at local electoral boundary levels, with a potential expansion to understanding how crucial indicators like income affect voting patterns within Singaporean electoral boundaries. This is **important** because such data was previously not available, for one reason or the other. 

Secondly, we have also successfully produced charts that compare income and education across electoral boundaries, as shown in the previous results section. By comparing all 31 electoral bounds against each other, we are able to get a big picture perspective on income and educational equity in Singapore, and how it may potentially vary. An observation I found from this is that areas with the highest percentage of high income (> $12,000) typically have higher education attainments too, such as the `HOLLAND-BUKIT TIMAH` constituency. However, I must note that the point of the project is not necessarily to make conjectures about how income/education may affect voting patterns. The chart and crosswalk are merely tools that can allow future researchers to reach a conclusion on voting patterns if desired. Any such insight now will be jumping to conclusions. 

Lastly, the R Shiny dashboard is extremely useful in terms of engaging individual Singaporeans to find out more about insights from their constituency. Typical Singaporeans will struggle to know what "Planning Area" they are a part of (such information is not really publically available unless one goes onto the data.gov portal to view the map boundaries manually). However, all adult Singaporeans know which constituency they are voting in, and thus can interactively look for data and link it to their immediate place or community. 

# Limitations, Future Work, Conclusion

The main limitation of this project is the fact that the crosswalk model can potentially be improved. In the methods section, we initially mentioned that we used an **areal weighting** method to construct intersection weights, instead of **population weighting**. There is a case to be made that a hybrid method that incorporates both (possibly by a set of weights) will be more rigorous than a strictly areal approach to constructing the crosswalk. 

Another limitation of this project is that visualizations on the map level were limited by the capabilities of R Shiny. It was initially in our interest to construct bar charts within the popups that show indiviudal electoral bounds's income/education data, but it was not possible in R. This is something that might be developed in future iterations of this project. 

In terms of future work, the future is bright. I am personally interested in fleshing out this project, in a few ways: 
+ Including different forms of data other than income and education data
+ Increasing the amount of visualization available (e.g. including bar charts within pop-ups)
+ Conducting statistical analysis of voting patterns based on a basket of variables
+ Identifying how variables change across temporal time (perhaps with the inclusion of a slider that can scroll through a timeline)

The point of this project was to provide tools for researchers to be able to transpose Singaporean planning area data to electoral boundary levels, which we have attained via the crosswalk. It is ultimately in our best interest that with such data available at that granularity, we can increase civic-political engagement by allowing locals to "interact" with our R Shiny dashboard, to allow them to get a better sense of the 'metrics' that define their place and immediate community. 

# Data Sources

[a] https://data.gov.sg/dataset/electoral-boundary_2020 

[b] https://data.gov.sg/dataset/master-plan-2014-planning-area-boundary-web?resource_id=2ab23cb2-b1a4-4b1a-a9e1-b9cad0ac159b 

[c] https://data.gov.sg/dataset/resident-population-aged-15-years-and-over-by-planning-area-and-highest-qualification-attained-2015 

[d] https://data.gov.sg/dataset/resident-working-persons-aged-15-years-over-by-planning-area-gross-monthly-income-from-work-2015 

[e] https://data.gov.sg/dataset/parliamentary-general-election-results 

# Sources 
[1] Rajan, S. T., Zhang, L. M., & Koh, F. (2020, September 2). GE2020 official results: PAP retains West Coast GRC with 51.69% of votes against Tan Cheng Bock's PSP. The Straits Times. https://www.straitstimes.com/politics/ge2020-sample-count-results-pap-leads-with-52-of-votes-in-west-coast-grc-against-tan-cheng. 

[2] Sim, D., & Jaipragas, B. (2020, June 27). PAP drops Ivan Lim as Singapore election candidate amid online backlash. South China Morning Post. https://www.scmp.com/week-asia/politics/article/3090873/online-backlash-forces-singapores-pap-drop-ivan-lim-election. 

[3] Tannen, J. (2021, March 2). R Tutorial: How I crosswalk Philadelphia's election data. sixtysixwards. https://sixtysixwards.com/home/crosswalk-tutorial/. 

[4] Eckert, F., Gvirtz, A., Liang, J., & Peters, M. (2020, February). A Method to Construct Geographical Crosswalks with an Application to US Counties since 1790. NBER Working Paper #26770, 2020. https://www.fpeckert.me/eglp/.

# License 
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# Contact
Please direct all questions about the project to josea [at] uchicago [dot] edu.
