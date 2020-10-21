COVID Project
================
Team Delta
2020-10-20

  - [Details on the datasets](#details-on-the-datasets)
      - [COVID dataset:](#covid-dataset)
      - [Interventions dataset:](#interventions-dataset)
      - [Combined dataset: *(Please read
        this)*](#combined-dataset-please-read-this)
  - [Combining Population and COVID
    Datasets](#combining-population-and-covid-datasets)
  - [Intervention Measures Dataset](#intervention-measures-dataset)
  - [Combining the COVID and Interventions
    Datasets](#combining-the-covid-and-interventions-datasets)
  - [Combining the Intervention Measures and COVID
    Datasets](#combining-the-intervention-measures-and-covid-datasets)

# Details on the datasets

<!-- -------------------------------------------------- -->

### COVID dataset:

To read the csv of the covid data, use this code:

``` r
filename_covid <- "./data/covid_data.csv"
df_covid <- read_csv(filename_covid)
```

    ## Warning: Missing column names filled in: 'X1' [1]

    ## Parsed with column specification:
    ## cols(
    ##   X1 = col_double(),
    ##   state = col_character(),
    ##   date = col_date(format = ""),
    ##   total_cases = col_double(),
    ##   total_deaths = col_double(),
    ##   total_population = col_double(),
    ##   cases_per100k = col_double(),
    ##   deaths_per100k = col_double()
    ## )

For each unique pair of **state** and **date**, it contains:

  - **total\_cases:** case count in number of people
  - **total\_deaths:** death count in number of people
  - **total\_population:** state population in number of people
  - **cases\_per100k:** case count per 100k people
  - **deaths\_per100k:** death count per 100k people

### Interventions dataset:

To read the csv of the intervention data, use this code:

``` r
filename_intv <- "./data/intervention_data.csv"
df_interventions <- read_csv(filename_intv)
```

    ## Warning: Missing column names filled in: 'X1' [1]

    ## Parsed with column specification:
    ## cols(
    ##   X1 = col_double(),
    ##   State = col_character(),
    ##   Date = col_date(format = ""),
    ##   Measure_L1 = col_character(),
    ##   Measure_L2 = col_character(),
    ##   Measure_L3 = col_character(),
    ##   Measure_L4 = col_character()
    ## )

Contains the following columns:

  - **State:** which state a measure was enacted in
  - **Date:** what day the measure was enacted
  - **Measure\_L1 through Measure\_L4:** A description of the measure,
    getting more specific as the number increases
      - There are 8 L1 measure categories, 66 L2 categories, and 642 L3
        categories. L4 has many ‘NA’ descriptions.

### Combined dataset: *(Please read this)*

To read the csv of the combined data, use this code:

``` r
filename_data <- "./data/combined_data.csv"
df_data <- read_csv(filename_data)
```

    ## Warning: Missing column names filled in: 'X1' [1]

    ## Parsed with column specification:
    ## cols(
    ##   X1 = col_double(),
    ##   state = col_character(),
    ##   date = col_date(format = ""),
    ##   total_cases = col_double(),
    ##   total_deaths = col_double(),
    ##   total_population = col_double(),
    ##   cases_per100k = col_double(),
    ##   deaths_per100k = col_double(),
    ##   Measures = col_character(),
    ##   `Resource allocation` = col_double(),
    ##   `Risk communication` = col_double(),
    ##   `Social distancing` = col_double(),
    ##   `Healthcare and public health capacity` = col_double(),
    ##   `Travel restriction` = col_double(),
    ##   `Case identification, contact tracing and related measures` = col_double(),
    ##   `Returning to normal life` = col_double(),
    ##   `Environmental measures` = col_logical(),
    ##   Num_measures_perday = col_double()
    ## )

For each unique pair of **state** and **date**, it contains:

  - **total\_cases:** case count in number of people
  - **total\_deaths:** death count in number of people
  - **total\_population:** state population in number of people
  - **cases\_per100k:** case count per 100k people
  - **deaths\_per100k:** death count per 100k people
  - **num\_measures\_perday** total number of measures enacted on that
    day
  - **measures:** a list of the types of L1 measures enacted that day,
    duplicates removed, separated by a semicolon
  - **the 8 L1 measure categories** the number of measures of that type
    were enacted on that day.
      - For example, 4 different risk communication measures were taken
        in Alabama on 2020-03-13.
      - If you want to get only whether that type of measure was taken,
        not how many, use \`measure name\` \> 0

#### Run this file again to pull the updated datasets from the New York Times and amel-github.

# Combining Population and COVID Datasets

<!-- -------------------------------------------------- -->

Load up the population data from the US Census Bureau

``` r
## Load the census bureau data with the following tibble name.
filename <- "./data/census_population_data.csv"
df_pop <- read_csv(filename, skip = 1)
```

    ## Parsed with column specification:
    ## cols(
    ##   id = col_character(),
    ##   `Geographic Area Name` = col_character(),
    ##   `Estimate!!Total` = col_double(),
    ##   `Margin of Error!!Total` = col_character()
    ## )

``` r
df_pop
```

    ## # A tibble: 3,221 x 4
    ##    id            `Geographic Area Name`  `Estimate!!Tota~ `Margin of Error!!Tot~
    ##    <chr>         <chr>                              <dbl> <chr>                 
    ##  1 0500000US010~ Autauga County, Alabama            55200 *****                 
    ##  2 0500000US010~ Baldwin County, Alabama           208107 *****                 
    ##  3 0500000US010~ Barbour County, Alabama            25782 *****                 
    ##  4 0500000US010~ Bibb County, Alabama               22527 *****                 
    ##  5 0500000US010~ Blount County, Alabama             57645 *****                 
    ##  6 0500000US010~ Bullock County, Alabama            10352 *****                 
    ##  7 0500000US010~ Butler County, Alabama             20025 *****                 
    ##  8 0500000US010~ Calhoun County, Alabama           115098 *****                 
    ##  9 0500000US010~ Chambers County, Alaba~            33826 *****                 
    ## 10 0500000US010~ Cherokee County, Alaba~            25853 *****                 
    ## # ... with 3,211 more rows

Pull the New York Times COVID case and death counts dataset

``` r
## The URL for the NYT covid-19 county-level data
url_nyt <- "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-counties.csv"
filename_nyt <- "./data/nyt_covid.csv"
curl::curl_download(
        url_nyt,
        destfile = filename_nyt
      )
df_nyt <- read_csv(filename_nyt)
```

    ## Parsed with column specification:
    ## cols(
    ##   date = col_date(format = ""),
    ##   county = col_character(),
    ##   state = col_character(),
    ##   fips = col_character(),
    ##   cases = col_double(),
    ##   deaths = col_double()
    ## )

``` r
df_nyt
```

    ## # A tibble: 647,924 x 6
    ##    date       county      state      fips  cases deaths
    ##    <date>     <chr>       <chr>      <chr> <dbl>  <dbl>
    ##  1 2020-01-21 Snohomish   Washington 53061     1      0
    ##  2 2020-01-22 Snohomish   Washington 53061     1      0
    ##  3 2020-01-23 Snohomish   Washington 53061     1      0
    ##  4 2020-01-24 Cook        Illinois   17031     1      0
    ##  5 2020-01-24 Snohomish   Washington 53061     1      0
    ##  6 2020-01-25 Orange      California 06059     1      0
    ##  7 2020-01-25 Cook        Illinois   17031     1      0
    ##  8 2020-01-25 Snohomish   Washington 53061     1      0
    ##  9 2020-01-26 Maricopa    Arizona    04013     1      0
    ## 10 2020-01-26 Los Angeles California 06037     1      0
    ## # ... with 647,914 more rows

Join the population and COVID datasets by their county code

``` r
## Create a `fips` column by extracting the county code
df_fips <- 
  df_pop %>%
  mutate(fips = substr(id, 10, 16)) %>%
  subset(select = -c(id))
df_fips
```

    ## # A tibble: 3,221 x 4
    ##    `Geographic Area Name`   `Estimate!!Total` `Margin of Error!!Total` fips 
    ##    <chr>                                <dbl> <chr>                    <chr>
    ##  1 Autauga County, Alabama              55200 *****                    01001
    ##  2 Baldwin County, Alabama             208107 *****                    01003
    ##  3 Barbour County, Alabama              25782 *****                    01005
    ##  4 Bibb County, Alabama                 22527 *****                    01007
    ##  5 Blount County, Alabama               57645 *****                    01009
    ##  6 Bullock County, Alabama              10352 *****                    01011
    ##  7 Butler County, Alabama               20025 *****                    01013
    ##  8 Calhoun County, Alabama             115098 *****                    01015
    ##  9 Chambers County, Alabama             33826 *****                    01017
    ## 10 Cherokee County, Alabama             25853 *****                    01019
    ## # ... with 3,211 more rows

``` r
## Join df_covid and df_pop by fips.
df_covid <- 
  left_join(df_nyt, df_fips, by = c("fips" = "fips"))
df_covid
```

    ## # A tibble: 647,924 x 9
    ##    date       county state fips  cases deaths `Geographic Are~ `Estimate!!Tota~
    ##    <date>     <chr>  <chr> <chr> <dbl>  <dbl> <chr>                       <dbl>
    ##  1 2020-01-21 Snoho~ Wash~ 53061     1      0 Snohomish Count~           786620
    ##  2 2020-01-22 Snoho~ Wash~ 53061     1      0 Snohomish Count~           786620
    ##  3 2020-01-23 Snoho~ Wash~ 53061     1      0 Snohomish Count~           786620
    ##  4 2020-01-24 Cook   Illi~ 17031     1      0 Cook County, Il~          5223719
    ##  5 2020-01-24 Snoho~ Wash~ 53061     1      0 Snohomish Count~           786620
    ##  6 2020-01-25 Orange Cali~ 06059     1      0 Orange County, ~          3164182
    ##  7 2020-01-25 Cook   Illi~ 17031     1      0 Cook County, Il~          5223719
    ##  8 2020-01-25 Snoho~ Wash~ 53061     1      0 Snohomish Count~           786620
    ##  9 2020-01-26 Maric~ Ariz~ 04013     1      0 Maricopa County~          4253913
    ## 10 2020-01-26 Los A~ Cali~ 06037     1      0 Los Angeles Cou~         10098052
    ## # ... with 647,914 more rows, and 1 more variable: `Margin of
    ## #   Error!!Total` <chr>

``` r
## Rename and down select columns
df_covid_data <-
  df_covid %>%
  select(
    date,
    county,
    state,
    fips,
    cases,
    deaths,
    population = `Estimate!!Total`
  ) %>%
  filter(population != 'NA')

df_covid_data
```

    ## # A tibble: 640,912 x 7
    ##    date       county      state      fips  cases deaths population
    ##    <date>     <chr>       <chr>      <chr> <dbl>  <dbl>      <dbl>
    ##  1 2020-01-21 Snohomish   Washington 53061     1      0     786620
    ##  2 2020-01-22 Snohomish   Washington 53061     1      0     786620
    ##  3 2020-01-23 Snohomish   Washington 53061     1      0     786620
    ##  4 2020-01-24 Cook        Illinois   17031     1      0    5223719
    ##  5 2020-01-24 Snohomish   Washington 53061     1      0     786620
    ##  6 2020-01-25 Orange      California 06059     1      0    3164182
    ##  7 2020-01-25 Cook        Illinois   17031     1      0    5223719
    ##  8 2020-01-25 Snohomish   Washington 53061     1      0     786620
    ##  9 2020-01-26 Maricopa    Arizona    04013     1      0    4253913
    ## 10 2020-01-26 Los Angeles California 06037     1      0   10098052
    ## # ... with 640,902 more rows

Aggregate by state

``` r
df_covid_bystate <-
  df_covid_data %>%
  group_by(state, date) %>%
  summarise(
    total_cases = sum(cases), 
    total_deaths = sum(deaths),
    total_population_sofar = sum(population)
    ) %>%
  ungroup() %>%
  group_by(state) %>%
  mutate(total_population = max(total_population_sofar)) %>%
  subset(select = -c(total_population_sofar))
```

    ## `summarise()` regrouping output by 'state' (override with `.groups` argument)

``` r
df_covid_bystate
```

    ## # A tibble: 11,994 x 5
    ## # Groups:   state [52]
    ##    state   date       total_cases total_deaths total_population
    ##    <chr>   <date>           <dbl>        <dbl>            <dbl>
    ##  1 Alabama 2020-03-13           6            0          4864680
    ##  2 Alabama 2020-03-14          12            0          4864680
    ##  3 Alabama 2020-03-15          23            0          4864680
    ##  4 Alabama 2020-03-16          29            0          4864680
    ##  5 Alabama 2020-03-17          39            0          4864680
    ##  6 Alabama 2020-03-18          51            0          4864680
    ##  7 Alabama 2020-03-19          78            0          4864680
    ##  8 Alabama 2020-03-20         106            0          4864680
    ##  9 Alabama 2020-03-21         131            0          4864680
    ## 10 Alabama 2020-03-22         157            0          4864680
    ## # ... with 11,984 more rows

Normalize the data per 100k persons

``` r
## Normalize cases and deaths
mult = 10e4
df_covid_norm <-
  df_covid_bystate %>%
  mutate(
    cases_per100k = total_cases/total_population*mult, 
    deaths_per100k = total_deaths/total_population*mult
    )

df_covid_norm
```

    ## # A tibble: 11,994 x 7
    ## # Groups:   state [52]
    ##    state date       total_cases total_deaths total_population cases_per100k
    ##    <chr> <date>           <dbl>        <dbl>            <dbl>         <dbl>
    ##  1 Alab~ 2020-03-13           6            0          4864680         0.123
    ##  2 Alab~ 2020-03-14          12            0          4864680         0.247
    ##  3 Alab~ 2020-03-15          23            0          4864680         0.473
    ##  4 Alab~ 2020-03-16          29            0          4864680         0.596
    ##  5 Alab~ 2020-03-17          39            0          4864680         0.802
    ##  6 Alab~ 2020-03-18          51            0          4864680         1.05 
    ##  7 Alab~ 2020-03-19          78            0          4864680         1.60 
    ##  8 Alab~ 2020-03-20         106            0          4864680         2.18 
    ##  9 Alab~ 2020-03-21         131            0          4864680         2.69 
    ## 10 Alab~ 2020-03-22         157            0          4864680         3.23 
    ## # ... with 11,984 more rows, and 1 more variable: deaths_per100k <dbl>

For each state and day, **df\_covid\_norm** contains:

  - **total\_cases:** case count in number of people
  - **total\_deaths:** death count in number of people
  - **total\_population:** state population in number of people
  - **cases\_per100k:** case count per 100k people
  - **deaths\_per100k:** death count per 100k people

<!-- end list -->

``` r
write.csv(df_covid_norm,'./data/covid_data.csv')
```

# Intervention Measures Dataset

<!-- -------------------------------------------------- -->

Pull the non-pharmaceutical intervention measures dataset

``` r
## The URL for the intervention measures country-level data
url_intv <- "https://raw.githubusercontent.com/amel-github/covid19-interventionmeasures/master/COVID19_non-pharmaceutical-interventions_version2_utf8.csv"
  
## Set the filename of the data to download
filename_intv <- "./data/amel-interventions.csv"

## Download the data locally
curl::curl_download(
        url_intv,
        destfile = filename_intv
      )

## Loads the downloaded csv
df_intv <- read_csv(filename_intv)
```

    ## Parsed with column specification:
    ## cols(
    ##   id = col_double(),
    ##   Country = col_character(),
    ##   iso3 = col_character(),
    ##   State = col_character(),
    ##   Region = col_character(),
    ##   Date = col_date(format = ""),
    ##   Measure_L1 = col_character(),
    ##   Measure_L2 = col_character(),
    ##   Measure_L3 = col_character(),
    ##   Measure_L4 = col_character(),
    ##   Status = col_character(),
    ##   Comment = col_character(),
    ##   Source = col_character()
    ## )

``` r
df_intv
```

    ## # A tibble: 6,271 x 13
    ##       id Country iso3  State Region Date       Measure_L1 Measure_L2 Measure_L3
    ##    <dbl> <chr>   <chr> <chr> <chr>  <date>     <chr>      <chr>      <chr>     
    ##  1     1 Albania ALB   Alba~ Alban~ 2020-02-25 Risk comm~ Educate a~ Encourage~
    ##  2     2 Albania ALB   Alba~ Alban~ 2020-02-25 Resource ~ Crisis ma~ Financial~
    ##  3     3 Albania ALB   Alba~ Alban~ 2020-02-25 Healthcar~ Adapt pro~ Implement~
    ##  4     4 Albania ALB   Alba~ Alban~ 2020-02-25 Case iden~ Airport h~ Specific ~
    ##  5     5 Albania ALB   Alba~ Alban~ 2020-03-08 Travel re~ Airport r~ Cancellat~
    ##  6     6 Albania ALB   Alba~ Alban~ 2020-03-08 Social di~ Closure o~ Complete ~
    ##  7     7 Albania ALB   Alba~ Alban~ 2020-03-08 Social di~ Closure o~ Complete ~
    ##  8     8 Albania ALB   Alba~ Alban~ 2020-03-08 Social di~ Closure o~ Complete ~
    ##  9     9 Albania ALB   Alba~ Alban~ 2020-03-08 Social di~ Small gat~ Complete ~
    ## 10    10 Albania ALB   Alba~ Alban~ 2020-03-08 Social di~ Mass gath~ Sport eve~
    ## # ... with 6,261 more rows, and 4 more variables: Measure_L4 <chr>,
    ## #   Status <chr>, Comment <chr>, Source <chr>

Tidy the intervention measures dataset

``` r
#df_intv # Raw data

df_interventions <-
  df_intv %>%
  # Filter to USA, take out USA totals, and remove the 18 non-state subregions ex. "Broward County"
  filter(iso3 == "USA", State != "United States of America", Region == State) %>%
  select(
    State,
    Date,
    Measure_L1,
    Measure_L2,
    Measure_L3,
    Measure_L4
  )

df_interventions
```

    ## # A tibble: 1,028 x 6
    ##    State  Date       Measure_L1  Measure_L2     Measure_L3        Measure_L4    
    ##    <chr>  <date>     <chr>       <chr>          <chr>             <chr>         
    ##  1 Alaba~ 2020-03-06 Resource a~ Activate or e~ Risk management ~ Set up crisis~
    ##  2 Alaba~ 2020-03-13 Risk commu~ Educate and a~ Encourage hand h~ <NA>          
    ##  3 Alaba~ 2020-03-13 Resource a~ Activate or e~ Declare state of~ <NA>          
    ##  4 Alaba~ 2020-03-13 Risk commu~ Educate and a~ Respiratory etiq~ Advice campai~
    ##  5 Alaba~ 2020-03-13 Risk commu~ Educate and a~ Respiratory etiq~ Advice campai~
    ##  6 Alaba~ 2020-03-13 Risk commu~ Educate and a~ Promote self-ini~ <NA>          
    ##  7 Alaba~ 2020-03-13 Social dis~ Mass gatherin~ Limit up to 500 ~ <NA>          
    ##  8 Alaba~ 2020-03-18 Social dis~ Mass gatherin~ Measures for ele~ Elections pos~
    ##  9 Alaba~ 2020-03-19 Social dis~ Small gatheri~ Limit up to 25 p~ <NA>          
    ## 10 Alaba~ 2020-03-19 Risk commu~ Educate and a~ Promote social d~ Promote the 2~
    ## # ... with 1,018 more rows

**df\_interventions** contains the following columns:

  - **State:** which state a measure was enacted in
  - **Date:** what day the measure was enacted
  - **Measure\_L1 through Measure\_L4:** A description of the measure,
    getting more specific as the number increases
      - There are 8 L1 measure categories, 66 L2 categories, and 642 L3
        categories. L4 has many ‘NA’ descriptions.

<!-- end list -->

``` r
write.csv(df_interventions,'./data/intervention_data.csv')
```

# Combining the COVID and Interventions Datasets

<!-- -------------------------------------------------- -->

We can’t join to the covid dataset this way because of the dates not
being distinct. YEt there are multiple measures taken on the same day,
so how to combine? First of all, make it more manageable by taking away
all designations except L1, the broadest one with only 8 categories.

``` r
df_intv2 <-
  df_interventions %>%
    select(
      State,
      Date,
      Measure = Measure_L1
    )

df_intv2
```

    ## # A tibble: 1,028 x 3
    ##    State   Date       Measure            
    ##    <chr>   <date>     <chr>              
    ##  1 Alabama 2020-03-06 Resource allocation
    ##  2 Alabama 2020-03-13 Risk communication 
    ##  3 Alabama 2020-03-13 Resource allocation
    ##  4 Alabama 2020-03-13 Risk communication 
    ##  5 Alabama 2020-03-13 Risk communication 
    ##  6 Alabama 2020-03-13 Risk communication 
    ##  7 Alabama 2020-03-13 Social distancing  
    ##  8 Alabama 2020-03-18 Social distancing  
    ##  9 Alabama 2020-03-19 Social distancing  
    ## 10 Alabama 2020-03-19 Risk communication 
    ## # ... with 1,018 more rows

Creating a wide version, to make sure each (State, Date) pair is unique
so that we can join with the covid dataset later.

``` r
df_wide_intv <-
  df_intv2 %>%
  group_by(State, Date, Measure) %>%
  # Remove repeat rows with same values for State, Date, and Measure
  filter(row_number(Measure) == 1) %>%
  
  pivot_wider(
    names_from = c("Measure"),
    values_from = Measure
  ) %>%
  unite(col = "measures", -c("Date", "State"), sep = "; ", na.rm = TRUE)

# In df_wide_intv Measures is all of the L1 designations concatenated with "; " between
df_wide_intv
```

    ## # A tibble: 408 x 3
    ## # Groups:   State, Date [408]
    ##    State   Date       measures                                                  
    ##    <chr>   <date>     <chr>                                                     
    ##  1 Alabama 2020-03-06 Resource allocation                                       
    ##  2 Alabama 2020-03-13 Resource allocation; Risk communication; Social distancing
    ##  3 Alabama 2020-03-18 Social distancing                                         
    ##  4 Alabama 2020-03-19 Resource allocation; Risk communication; Social distancin~
    ##  5 Alabama 2020-03-20 Social distancing                                         
    ##  6 Alabama 2020-03-21 Resource allocation                                       
    ##  7 Alabama 2020-03-23 Resource allocation                                       
    ##  8 Alabama 2020-03-26 Resource allocation; Social distancing                    
    ##  9 Alabama 2020-03-28 Social distancing                                         
    ## 10 Alabama 2020-03-30 Risk communication                                        
    ## # ... with 398 more rows

``` r
df_wide_intv2 <-
  df_intv2 %>%
  group_by(State, Date, Measure) %>%
  summarize(num_measures = n()) %>%
  pivot_wider(
    names_from = "Measure",
    values_from = num_measures
  )
```

    ## `summarise()` regrouping output by 'State', 'Date' (override with `.groups` argument)

``` r
df_wide_intv2$num_measures_perday <- 
  cbind(
    rowSums(df_wide_intv2 %>%
      subset(select = -c(State, Date)), 
    na.rm = TRUE)
    )

# In df_wide_intv2 Num_measures_perday is the total num measures enacted on that day, and the 8 new columns are 
# the 8 possible L1 types of measures. The number stored is how many of that type were enacted on that day
df_wide_intv2
```

    ## # A tibble: 408 x 11
    ## # Groups:   State, Date [408]
    ##    State Date       `Resource alloc~ `Risk communica~ `Social distanc~
    ##    <chr> <date>                <int>            <int>            <int>
    ##  1 Alab~ 2020-03-06                1               NA               NA
    ##  2 Alab~ 2020-03-13                1                4                1
    ##  3 Alab~ 2020-03-18               NA               NA                1
    ##  4 Alab~ 2020-03-19                1                1                5
    ##  5 Alab~ 2020-03-20               NA               NA                5
    ##  6 Alab~ 2020-03-21                1               NA               NA
    ##  7 Alab~ 2020-03-23                1               NA               NA
    ##  8 Alab~ 2020-03-26                1               NA                2
    ##  9 Alab~ 2020-03-28               NA               NA                5
    ## 10 Alab~ 2020-03-30               NA                1               NA
    ## # ... with 398 more rows, and 6 more variables: `Healthcare and public health
    ## #   capacity` <int>, `Travel restriction` <int>, `Case identification, contact
    ## #   tracing and related measures` <int>, `Returning to normal life` <int>,
    ## #   `Environmental measures` <int>, num_measures_perday[,1] <dbl>

``` r
df_interventions <- 
  left_join(df_wide_intv, df_wide_intv2, by = c("State" = "State", "Date" = "Date"))
df_interventions
```

    ## # A tibble: 408 x 12
    ## # Groups:   State, Date [408]
    ##    State Date       measures `Resource alloc~ `Risk communica~ `Social distanc~
    ##    <chr> <date>     <chr>               <int>            <int>            <int>
    ##  1 Alab~ 2020-03-06 Resourc~                1               NA               NA
    ##  2 Alab~ 2020-03-13 Resourc~                1                4                1
    ##  3 Alab~ 2020-03-18 Social ~               NA               NA                1
    ##  4 Alab~ 2020-03-19 Resourc~                1                1                5
    ##  5 Alab~ 2020-03-20 Social ~               NA               NA                5
    ##  6 Alab~ 2020-03-21 Resourc~                1               NA               NA
    ##  7 Alab~ 2020-03-23 Resourc~                1               NA               NA
    ##  8 Alab~ 2020-03-26 Resourc~                1               NA                2
    ##  9 Alab~ 2020-03-28 Social ~               NA               NA                5
    ## 10 Alab~ 2020-03-30 Risk co~               NA                1               NA
    ## # ... with 398 more rows, and 6 more variables: `Healthcare and public health
    ## #   capacity` <int>, `Travel restriction` <int>, `Case identification, contact
    ## #   tracing and related measures` <int>, `Returning to normal life` <int>,
    ## #   `Environmental measures` <int>, num_measures_perday[,1] <dbl>

# Combining the Intervention Measures and COVID Datasets

<!-- -------------------------------------------------- -->

``` r
df_combined_data <- 
  left_join(df_covid_norm, df_interventions, by = c("state" = "State", "date" = "Date"))
df_combined_data
```

    ## # A tibble: 11,994 x 17
    ## # Groups:   state [52]
    ##    state date       total_cases total_deaths total_population cases_per100k
    ##    <chr> <date>           <dbl>        <dbl>            <dbl>         <dbl>
    ##  1 Alab~ 2020-03-13           6            0          4864680         0.123
    ##  2 Alab~ 2020-03-14          12            0          4864680         0.247
    ##  3 Alab~ 2020-03-15          23            0          4864680         0.473
    ##  4 Alab~ 2020-03-16          29            0          4864680         0.596
    ##  5 Alab~ 2020-03-17          39            0          4864680         0.802
    ##  6 Alab~ 2020-03-18          51            0          4864680         1.05 
    ##  7 Alab~ 2020-03-19          78            0          4864680         1.60 
    ##  8 Alab~ 2020-03-20         106            0          4864680         2.18 
    ##  9 Alab~ 2020-03-21         131            0          4864680         2.69 
    ## 10 Alab~ 2020-03-22         157            0          4864680         3.23 
    ## # ... with 11,984 more rows, and 11 more variables: deaths_per100k <dbl>,
    ## #   measures <chr>, `Resource allocation` <int>, `Risk communication` <int>,
    ## #   `Social distancing` <int>, `Healthcare and public health capacity` <int>,
    ## #   `Travel restriction` <int>, `Case identification, contact tracing and
    ## #   related measures` <int>, `Returning to normal life` <int>, `Environmental
    ## #   measures` <int>, num_measures_perday[,1] <dbl>

``` r
write.csv(df_combined_data,'./data/combined_data.csv')
```
