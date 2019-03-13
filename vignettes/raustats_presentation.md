---
title: "`raustats` -- Downloading ABS & RBA data with R"
author: "David Mitchell"
date: "2019-02-07"
output:
  powerpoint_presentation:
    slide_level: 2  # Use this to override the default slide title 
---

<!-- RMarkdown references -->
<!-- https://support.rstudio.com/hc/en-us/articles/200486468-Authoring-R-Presentations -->
<!-- https://support.rstudio.com/hc/en-us/articles/360004672913-Rendering-PowerPoint-Presentations-with-RStudio -->


# Introduction

## Introduction

- `raustats` provides functions to search and download ABS & RBA statistics
- Main features:
  * Access to all time series statistics in the ABS statistical catalogue and
    the RBA website
	* Access to ABS cross-section statistical catalogue data to be added
  * Access to statistics available through the ABS Beta API:
    [ABS.Stat](http://stat.data.abs.gov.au)
  * Returns data in long (or tidy) format for direct integration with modern R
    tools: `ggplot2`, `tidyverse`, etc.
  * Support for regular expression (`grep`) style searches for data sets.

- Inspired by R packages: `wbstats`, `OECD` `IMFData`, `imfr` - API access to
  World Bank, OECD & IMF API data 

::: notes
This is a speaker note.

- Use basic Markdown 
- like this list
- *and inline formatting*
:::

<!-- You can use the *Two Content* layout to put material in side by side
columns. For example, you can put text next to an image on the same slide. To
use the *Two Content* layout, nest two div containers of class column inside one
div container of class columns.

::::: {.columns}
::: {.column width="40%"}
contents...
:::
::: {.column width="60%"}
contents...
:::
:::::

-->


## Quick-start

- Load the library:

```r
library(raustats)

s <- c(55, 65)
x <- c(1.0, 1.5)
lnk.avg <- mean(s)
lng.avg <- sum(s*x)/sum(x)

time <- x/s
sum(time)

2.5/sum(time)

```

- Downloading latest Consumer Price Index (CPI) catalogue (6401.0) data as simple as:

```r
cpi_all <- abs_cat_stats("6401.0")
```

- or via [ABS.Stat](http://stat.data.abs.gov.au/):

```r
cpi_api <- abs_stats("CPI", filter=list(MEASURE=1, REGION=c(1:8,50),
                                        INDEX=10001, TSEST=10, FREQUENCY="Q"))
```

- Library also includes RBA statistical data access functions, e.g. RBA assets
  and liabilities (RBA Statistical Table A1) can be downloaded with:

```r
rba_bs <- rba_stats("A1")
```


#  ABS Catalogue statistics functions

## ABS catalogue statistics functions

- Two core functions:

  * `abs_cat_stats` -- download specified ABS Catalogue tables
  * `abs_cat_tables` -- list specified ABS Catalogue tables
  
- Several helper functions (called by `abs_cat_stats` and `abs_cat_tables`):

  * `abs_read_tss` -- extracts data from standard-formatted ABS Catalogue
	   time series spreadsheets.
  * `abs_download_data` -- downloads and saves ABS Catalogue tables from a
	   supplied URL. 
  * `abs_unzip_files` --  extracts Excel files from compressed ABS zip archives.

<!-- 
The helper functions are called by the core functions and should generally not
need to be called directly by users---though there are some cases where these
functions may be useful.
-->


## The `abs_cat_stats` function

- By default, `abs_cat_stats` downloads all tables from the latest edition.
- Limit to specified tables with `tables` argument (default: `tables="all"`).
  - Releases accepts regular expressions.
- Select one or more issues with `releases` argument (default:
  `releases="Latest"`).
  
- E.g. downloads only Tables 1 and 2 from Catalogue no. 5206.0:

```r
ana_q <- abs_cat_stats(cat_no = "5206.0", tables=c("^Table 1\\.", "^Table 2\\."))
```

- or select tables by regular expression, matching one or more table names:


- return Table 1 from the December 2016 and 2017 releases of the quarterly
  national accounts:

```r
ana_Q4 <- abs_cat_stats(cat_no="5206.0", tables="Table 1", releases=c("Dec 2017","Dec 2016"));
```


## Listing ABS Catalogue tables with `abs_cat_tables`

- The `abs_cat_tables` function returns a list of all tables for one or more
  specified ABS Catalogue numbers. 
  
- List all available tables for the Catalogue nos. 5206.0 and 6401.0:


```r
ana_tables <- abs_cat_tables(cat_no="5206.0")
```

```r
## CPI
cpi_tables <- abs_cat_tables(cat_no="6401.0")
```

- Other arguments:
  * `releases`
  * `types` - one or more of 'tss', 'css' and 'pub'
  * `include_urls` - TRUE or FALSE






- Example: return all downloadable Data Cubes for Australian Statistical
Geography Standard (ASGS) main structure classification 
and digital boundaries (Catalogue no. 1270.0.55.001).


```r
asgs_files <- abs_cat_tables(cat_no="1270.0.55.001", types="css", include_urls=TRUE)
```




#  ABS.Stat access functions

##  ABS.Stat statistics access functions

- Two core functions:

  * `abs_datasets` --  list of all datasets available through
      [ABS.Stat](http://stat.data.abs.gov.au/)
  * `abs_search` --  searches for datasets matching the specified regular expression
  * `abs_stats` -- returns data from specified datasets.
  
- Several information functions:

  * `abs_dimensions` -- lists all available dimensions for specified dataset. 
	   

<!-- 
The helper functions are called by the core functions and should generally not
need to be called directly by users---though there are some cases where these
functions may be useful.
-->


##  ABS.Stat function examples

<!-- UP TO HERE -->




### Finding available data with `abs_datasets`

The `abs_datasets` function returns a. The function has two arguments: `lang`
(default is English: `lang="en"`) and `include_notes` (default:
`include_notes=FALSE`). The following example shows the results with notes
included.


```r
datasets <- abs_datasets();
head(datasets)
```


### Cached list of available datasets `abs_cachelist`

For performance, a cached list of datasets available through the
[ABS.Stat](http://stat.data.abs.gov.au/) API is provided in the `abs_cachelist`
data set included with `raustats`.  `abs_cachelist` is the default source used
in `abs_search()` and `abs_stats()` to find matching ABS datasets. 

By default, `abs_cachelist` is in English. To search indicators in a different
language, you can download an updated copy of `abs_cachelist` using
`abs_cache()` (next).


### Accessing updated available data with `abs_cache()`

Current information about datasets available from the
[ABS.Stat](http://stat.data.abs.gov.au/) API can be obtained by calling
`abs_cache`.^[`abs_cachelist` is simply a saved return of `abs_cache(lang =
"en")`.]  Updated cache information can be used in the `abs_search` and
`abs_stats` functions by setting the `cache` argument equal to the saved results
returned by `abs_cache`. While using updated cache information will ensure you
identify all available datasets, the `abs_cache` function can take a long time
to download all datasets, so use judiciously. Because of this, `abs_cache`
includes functionality to report progress back to the console. The frequency of
reporting can be set with the `progress` argument (by default `progress=10`).
The second example downloads a new cache in French.^[Note, the
[ABS.Stat](http://stat.data.abs.gov.au/) API currently has limited support for
other languages, so data may not be available in languages other than English.]


```r
## Default language: English
new_cache <- abs_cache(progress=10);
## French language version
new_cache_fr <- abs_cache(lang="fr", progress=10);
```


### Checking dataset dimensions with `abs_dimensions()`

The `abs_dimensions()` functions lists the name of all available dimensions and
the respective dimension type. Typical dimension types are: 'Dimension',
'TimeDimension' and 'Attribute'. 'Dimension' attributes are used in the `filter`
argument of `abs_stats` function. The following example lists the data
dimensions of the 'CPI' dataset.


```r
abs_dimensions('CPI');
```


A list of all available dimension codes and descriptions for a particular
dataset can be viewed by selecting the relevant dataset from `abs_cachelist` or
an updated cache list returned by `abs_cache`.


```r
str(abs_cachelist[["CPI"]]);
```


### Search available data with `abs_search()`

The `abs_search` function essentially has two modes of operation: 

  i) 'dataset search' mode, and
  ii) 'indicator search' mode.

In dataset search mode, the function searches and returns datasets matching the
specified regular expression. The following examples demonstrate use of the
function to find ABS datasets relating to the CPI and labour force. The
`code_only` argument specifies whether the function returns all information or
just the matching dataset identifiers.



```r
abs_search("CPI|consumer price index");

abs_search("CPI|consumer price index", code_only=TRUE);
```


```r
abs_search("labour force");

abs_search("^labour force$");
```

In indicator search mode, the function searches through all dimensions of a
specific dataset and returns a list of dimensions and dimension contents
matching all the provided regular expressions. The following examples
demonstrates use of the function to find indicators within the CPI data set.




```r
abs_search(c("All groups CPI","Sydney"), dataset="CPI");
```

If `code_only=TRUE`, the indicator search function returns only codes for each
matching dimension. This, in turn, can be used directly as input to the `filter`
argument of the `abs_stats` function. The following example returns dimension
codes for datasets matching both the regular expressions "All groups CPI" and
"Sydney".




```r
abs_search(c("All groups CPI","Sydney"), dataset="CPI", code_only=TRUE);
```


### Downloading data with `abs_stats()`

The `abs_stats()` function returns data from specified datasets available via
the [ABS.Stat](http://stat.data.abs.gov.au/) API. The following section outlines
typical use of the `abs_stats()` function, and also describes each of the core
function arguments.

The following example downloads original  All groups CPI index numbers for each
of the eight Australian state and territory capital cities and also the average
for all capital cities.


```r
cpi <- abs_stats(dataset="CPI", filter=list(MEASURE=1, REGION=c(1:8,50),
                                            INDEX=10001, TSEST=10, FREQUENCY="Q"));
```

The filter conditions are:

  * `MEASURE=1` -- 'Index Numbers'
  * `REGION=c(1:8,50)` -- Each of the eight capital cities (1--8) and all eight
    capital cities (50)
  * `INDEX=10001` -- 'All groups CPI'
  * `TSEST=10` -- 'Original' observations
  * `FREQUENCY=Q` -- Quarterly observations



The `filter` argument can also be set equal to "all", in which case the function
will attempt to download all observations available for the specified
dataset. However, if the dataset is large it may breach the
[ABS.Stat](http://stat.data.abs.gov.au/) API query length, record and/or session
time constraints.^[[ABS.Stat](http://stat.data.abs.gov.au/) has a query string
length limit (maximum of 1000 characters) on URL queries, a record return limit
(1 million records) and session time limits (maximum 10-minute session time
limit). (See the [ABS.Stat Web Services User Guide
FAQ](http://www.abs.gov.au/ausstats/abs@.nsf/Lookup/by%20Subject/1407.0.55.002~User%20Guide~Main%20Features~Frequently%20Asked%20Questions~9)
for further details.)] Queries that breach these limits will need to be
re-specified as multiple separate calls to obtain the required data.

For example, the following `abs_stats` function call, attempts to download all
series available for the CPI dataset, but the specified query length (1191
characters) exceeds maximum request URL character limit.


```r
cpi <- abs_stats(dataset="CPI", filter="all");
```

By default, `abs_stats` checks whether the query string length and the estimated
number of records to be returned and will halt execution if the query breaches
any of these conditions. Setting the `enforce_api_limits = FALSE` (default:
`TRUE`) will ignore these checks and submit the query to the
[ABS.Stat](http://stat.data.abs.gov.au/) API anyway. If the query fails it will
return an error. For example, the following function call will return an
error.


```r
cpi <- abs_stats(dataset="CPI", filter="all", enforce_api_limits=FALSE);
#> lexical error: invalid char in json text.
#>                                        http://stat.data.abs.gov.au/SDM
#>                      (right here) ------^
```

Setting the `return_url = TRUE` (default: `FALSE`) will return the RESTful URL
query string, but does not submit the query to the
[ABS.Stat](http://stat.data.abs.gov.au/) API, see the following example function
call and output.


```r
abs_stats(dataset="CPI", filter=list(MEASURE=1, REGION=c(1:8,50),
                                     INDEX=10001, TSEST=10, FREQUENCY="Q"),
          return_url=TRUE);
```

The `abs_search` function can be used to specify the filter. For example, the
following code block produces the same filter list, specified in the previous
example, and can subsequently be supplied to the `abs_stats` `filter` argument.



```r
filter_lst <- abs_search(c("Index numbers", "All groups",
                           "Sydney|Melbourne|Brisbane|Adelaide|Perth|Hobart|Darwin|Canberra|capital cities",
                           "Original", "Quarterly"),
                         dataset="CPI", code_only=TRUE);
cpi <- abs_stats("CPI", filter = filter_lst);
```

Users can also limit the date range to return by specifying one or
both`start_date` and `end_date` arguments. These accept either a numeric or
character arguments---if numeric it must be a four-digit year. If a character
string it can be either a monthly, quarterly, half-year or financial year as
formatted as follows: month -- '2016-M01', quarter -- '2016-Q1', half-year --
'2016-B2', financial year -- '2016-17'.  The following example returns all CPI
observations between September 2015 and June 2018.


```r
cpi <- abs_stats(dataset="CPI", filter=filter_lst,
                 start_date = "2015-Q3", end_date = "2018-Q2");
```

The other arguments `dimensionAtObservation` and `detail` provide refinements to
the URL query. These need not be modified by the user---the defaults are:
`dimensionAtObservation='AllDimensions` and `detail='Full'`.


# RBA statistics access functions {#rba-stats-functions}

### Finding available RBA data tables with `rba_table_cache`

The `rba_table_cache` function returns a dataset of all available RBA
statistical tables. The function scans the RBA website and returns a list of all
[Statistical tables](https://www.rba.gov.au/statistics/tables/), [Historical
data tables](https://www.rba.gov.au/statistics/historical-data.html) and
[Discontinued data
tables](https://www.rba.gov.au/statistics/discontinued-data.html). (The
`rba_tablecache` data set included in this package contains a pre-saved list of
all available RBA statistical tables.)

The dataset has four columns:

  * table_code
  * table_name
  * table_type -- one of either statistical table, historical data or
    discontinued data.
  * url.

The `rba_table_cache` function has no arguments. The following example shows use
of the function and the returned output.


```r
rba_cache <- rba_table_cache()
```



### Search available data with `rba_search()`

The `rba_search` function scans the RBA table cache for statistical tables
matching a regular expression supplied to `pattern`.  The `fields` argument
specifies the fields in the RBA table cache to search---by default the function
searches the `table_code` and `table_name` fields only. The `ignore.case`
argument allows for case sensitive regular expression matching---the default is
case insensitive matching (`ignore.case=TRUE`). The `cache` argument specifies
the RBA table cache to be used. If omitted, `rba_tablecache` is used.



```r
rba_search(pattern = "Liabilities and Assets");
```


### Accessing RBA statistical tables with `rba_stats()`

As previously outlined in the Quick Start section, the `rba_stats()` function
provides easy access to RBA statistical tables.  The `rba_stats()` function has
three mutually-exclusive table selection arguments: `table_no`, `pattern`, and
`url`, for selecting RBA statistical tables.  Specifying `table_no` selects
tables matching the specified RBA table number, e.g.  A1, B1, B11.1.  Specifying
`pattern` selects all RBA tables matching the regular expression specified in
`pattern`.  The `url` argument can be used to specify one or more valid RBA
statistical table URLs.

The function returns a long-format (tidy) table with the following columns:

  * *series_id* -- RBA series identifier
  * *date* -- Date-format date
  * *value* -- observation value
  * *title* -- data item name
  * *description* -- data item description
  * *frequency* -- Frequency (e.g. Annual, Quarterly, Monthly, Weekly, Daily)
  * *type* -- series type, one of Original, Trend, Seasonally Adjusted
  * *units* -- unit of measure (e.g. Percent, $ Millions, Index Numbers, Proportion)
  * *source* -- data source organisation
  * *data_type* -- data type (e.g. Original, Derived)
  * *table_no* -- RBA statistics table number (if available)
  * *table_title* -- RBA statistics table title.


The following examples demonstrate typical uses of the function and the various
function arguments. The first example downloads RBA Statistical Table A1 --
Liabilities and Assets using `table_no`.


```r
rba_a1 <- rba_stats(table_no = "A1")
```


The second example downloads the same tables using the `pattern` argument to
download tables matching the regular expression: 'Liabilities and Assets.+A1'.


```r
rba_a1 <- rba_stats(pattern = "Liabilities and Assets.+Summary")
```

And the third example downloads the same tables using the relevant URLs. The
example presented below first returns a list of all RBA tables matching the
supplied regular expression ('Liabilities and Assets.+A1') and then uses the
returned URLs to download each data set.


```r
a1_tables <- rba_search(pattern = "Liabilities and Assets.+Summary")
rba_a1 <- rba_stats(url = a1_tables$url)
```


Again, the `cache` argument specifies the RBA table cache to be used---if
omitted, `rba_tablecache` is used. The `rba_stats` function also optionally
accepts `rba_search` arguments: `fields` and `ignore.case`, for informing
pattern matching.

At present, the `rba_stats` function only handles [Statistical
tables](https://www.rba.gov.au/statistics/tables/), which have a consistently
structured format comprising full metadata and observations. Functionality to
handle [Historical data
tables](https://www.rba.gov.au/statistics/historical-data.html) and
[Discontinued data
tables](https://www.rba.gov.au/statistics/discontinued-data.html) are in
progress.



### Other RBA helper functions

There are also two RBA helper functions that are called by
`rba_stats`---that download and parse the RBA statistical
tables. These additional functions should not ordinarily need to be
called directly, but there may be situations in which users wish to
access these functions directly. The functions are:

  * `rba_file_download`
  * `rba_read_tss`

The following examples illustrate the use of these functions.

The `rba_download` function downloads and saves RBA tables from a supplied URL,
and returns a character vector of the saved local filenames. It is called inside
the `rba_stats` function and can be used to directly download one or more RBA
statistical table files. It is most usefully used in conjunction with the
`rba_search` function.



```r
a1_tables <- rba_search(pattern = "Liabilities and Assets.+Summary")
downloaded_tables <- rba_file_download(a1_tables$url, exdir=tempdir())
print(downloaded_tables)
```

The `rba_read_tss` function extracts data from standard-format RBA statistical
tables and returns it as a long-format (tidy) data frame.  It is also called by
the `rba_stats` function.  The simple example below shows use of the function to
extract data from the previously downloaded tables.


```r
a1_data <- rba_read_tss(downloaded_tables);
```




# Using data returned by `raustats`

Statistics returned by the `raustats` package functions are generally in long
(tidy) format data frames. This provides for easy integration with packages like
`ggplot2`, `tidyr` and `dplyr`. The following example illustrates how data
downloaded using the `raustats` can be easily transformed and plotted. This
example uses data from ABS' *Private New Capital Expenditure and Expected
Expenditure* (ABS Catalogue no. 5265.0).

First download selected tables from Catalogue no. 5265.0.

```r
capex_q <-
  abs_cat_stats("5625.0",
                tables=c("Actual Expenditure by Type of Asset and Industry - Current Prices",
                         "Actual Expenditure, By Type of Industry - Chain Volume Measures",
                         "Actual and Expected Capital Expenditure by Industry.+:Current Prices"));
```

Then add a new variable denoting Australian state/territory.


```r
library(dplyr)
## Add state/territory variable
capex_q <- capex_q %>%
  mutate(state = sub(sprintf(".*(%s).*",
                             paste(c("New South Wales","Victoria","Queensland","South Australia",
                                     "Western Australia","Tasmania","Northern Territory",
                                     "Australian Capital Territory","Total \\(State\\)"),
                                   collapse="|")),
                     "\\1", data_item_description, ignore.case=TRUE));
```

Finally, plot quarterly time series mining sector capital expenditure (at
current prices) by Australian state and territory using `ggplot`.


```r
library(ggplot2)
## Filter mining capital expenditure
capex_q_min <- capex_q %>%
  filter(grepl("mining", data_item_description, ignore.case=TRUE)) %>%
  filter(grepl("actual", data_item_description, ignore.case=TRUE)) %>%
  filter(grepl("current price", data_item_description, ignore.case=TRUE)) %>%
  filter(grepl("Total \\(Type of Asset.+\\)", data_item_description, ignore.case=TRUE))

ggplot(data=capex_q_min) +
  geom_line(aes(x=date, y=value/10^3, colour=state)) +
  scale_x_date(date_labels="%b\n%Y") +
  scale_y_continuous(limits=c(0, NA)) +
  labs(title="Australian mining sector capital expenditure, by state",
       y="Capital expenditure ($ billion)", x=NULL) +
  guides(colour = guide_legend(title=NULL)) + 
  theme(plot.title = element_text(hjust=0.5),
        legend.box = "horizontal",
        legend.position = "bottom",
        axis.text.x=element_text(angle=0, size=8));
```


# Unresolved issues

There are a few behaviours of the [ABS.Stat](http://stat.data.abs.gov.au/) API
that may help explain any unexpected results. As the
[ABS.Stat](http://stat.data.abs.gov.au/) API is still in Beta release and
subject to revision, some of these issues will be addressed in future versions
of the `raustats` package, as [ABS.Stat](http://stat.data.abs.gov.au/)
transitions towards a stable release version.


## Searching in other languages

The [ABS.Stat](http://stat.data.abs.gov.au/) API (Beta version) includes scope
to cater to multiple languages. However, at the time of writing, French appears
to be the only other language included in the data sets. Moreover, the text for
many records that are denoted as French appear to be in English. The ABS API
calling functions included in this library allow users to specify their
preferred language, but this functionality has been little tested to date.


## Query, data and session constraints

As already noted, the [ABS.Stat](http://stat.data.abs.gov.au/) API has a query
string length limit (1000 characters) and record return limit (1 million
records). We have not tested how rigorously these limits are
enforced. Accordingly, the `abs_stats` function includes the
`enforce_api_limits` argument to catch these limits before the query is
submitted. The argument also allows the user to override the limits and submit
the query anyway. This argument may be deprecated in future versions.

The [ABS.Stat](http://stat.data.abs.gov.au/) API also has a 10-minute session
time limit for users to download datasets via the SDMX service. The functions in
`raustats` almost exclusively use the SDMX-JSON query interface, so it is not
clear whether the session time limit applies. If time limits are an issue, the ABS
advises users to submit multiple smaller requests to retrieve the required data.


# Resources

  * [ABS.Stat -- http://stat.data.abs.gov.au/](http://stat.data.abs.gov.au/)
  * [ABS.Stat FAQ](http://stat.data.abs.gov.au/)
  
