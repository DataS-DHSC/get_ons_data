# get_ons_data
Discussion on obtaining data from ONS for downstream use.

![ons_screenshot](https://github.com/DataS-DHSC/get_ons_data/blob/master/ons_screenshot.jpeg)

## Options

There are a number of routes to obtain data from ONS to feed a [reproducible analytical pipeline](https://ukgovdatascience.github.io/rap-website/). Here we discuss some of the most useful options.

### API 

An [application programming interface](https://en.wikipedia.org/wiki/Application_programming_interface) (api) is a computing interface which defines interactions between multiple software intermediaries. It defines the kinds of calls or requests that can be made, how to make them, the data formats that should be used, the conventions to follow, etc.

ONS have an API, part of a service called Customise My Data (CMD). It currently contains a subset of ONS data. They have a [full list](https://onsdigital.github.io/dp-prototypes/prototypes/cmd-dataset-list/index.html) and guidance](https://developer.beta.ons.gov.uk/). The data is available in both .CSV or .XLSX format. Data may be in a more raw than the publication files, so may require some light processing. 

The most popular r package for calling APIs is [jsonlite](https://cran.r-project.org/web/packages/jsonlite/index.html). The documentation includes some [examples](https://cran.r-project.org/web/packages/jsonlite/vignettes/json-apis.html) of how to make calls using the package.

Advantages of the method are as follows:
- Flexible, efficient and easy to automate obtaining the data. 
- Easier than alternatives to build into an updatable pipleine. 
- Transparently available to government departments for free. No legal considerations.

Disadvantage are:
- Not all of ONS data is available. It may not be an option in some circumstances.
- The service is in beta so may not be perfect.
- It doens't guarantee a carbon copy of what is seen on the publication page. May be issues of timeliness, quality etc.

#### Example
Here we explore Covid deaths for a particular local authority, pulling data directly from the API.

``` r
library(tidyverse)
library(jsonlite)

dataset_url <- jsonlite::fromJSON('https://api.beta.ons.gov.uk/v1/datasets/weekly-deaths-local-authority')

#get latest version
latest_version <- dataset_url$links$latest_version$href

#select parameters for dimensions
time <- 2020
week <- '*'
geography <- 'E08000003'
causeofdeath <- 'covid-19'
placeofdeath <- 'hospital'
registrationoroccurrence <- 'registrations'

#build API call
api_call <- paste0(latest_version,'/observations?time=',time,'&week=',week,'&geography=',geography,'&causeofdeath=',causeofdeath,'&placeofdeath=',placeofdeath,'&registrationoroccurrence=',registrationoroccurrence)

#call API
api_results <- jsonlite::fromJSON(api_call)

#get into one df
api_results_df <- data.frame(Week = api_results$observations$dimensions$week$label, Count = as.numeric(api_results$observations$observation))
api_results_df$Week <- gsub('Week ','',api_results_df$Week)

#write subtitle
subtitle <- paste0('Local authority: ',geography,'\n',
                   'Cause of death: ',str_to_title(causeofdeath),'\n',
                   'Place of death: ',str_to_title(placeofdeath),'\n',
                   'Registration or occurrence: ',str_to_title(registrationoroccurrence))


#plot line chart
ggplot() + geom_line(data = api_results_df, aes(x = Week, y = Count), group = 1) + 
  geom_point(data = api_results_df, aes(x = Week, y = Count), group = 1) + 
  theme_minimal() + 
  labs(title = 'Weekly deaths, 2020', 
                         subtitle = subtitle, 
                         caption = 'Source: Office for National Statistics API')
```

## Download from direct link
To follow

## Web scrape from publication page
To follow
