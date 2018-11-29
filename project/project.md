Final Project
=============

### Due date: Dec 12, 2018 @ 11:59pm

Introduction
------------

-   The goal of this project is to access and analyze big data that cannot be loaded directly to R's base memory. The data you are going to analyze consists of flight arrival and departure details for all commercial flights within the USA, from October 1987 to April 2008. This is a large dataset: there are nearly 120 million records in total, and takes up 1.6 gigabytes of space compressed and 12 gigabytes when uncompressed.

-   Because of the overwhelming size of the data, we use a relational database (RDB) to efficiently manage the large dataset. In this project you are supposed to construct a database by using the `data.table` and `RSQLite` packages. Then you use `dplyr` to access the database and other R functions to analyze the data.

-   The project consists of three parts. In Part 1, you construct a SQLite database using the `DBI` and `RSQLite` packages. Part 2 provides basic questions. You must be able to solve these question using the knowledge learned in class. You will get full credit if you successfully complete parts 1 and 2. Part 3 contains extra questions. You are supposed to design your own analysis in a way that you think is reasonable, conduct the analysis, and interpret the outcomes. You will receive extra credits if you fulfill the request.

The dataset
-----------

-   The following website <http://stat-computing.org/dataexpo/2009/the-data.html> provides flights data between 1987-2008 in the form of bz2 compressed csv file format.

-   The data comes originally from [BTS](http://www.transtats.bts.gov/OT_Delay/OT_DelayCause1.asp) where it is described [in detail](http://www.transtats.bts.gov/Fields.asp?Table_ID=236).

-   Variable descriptions are as follows:

<!-- -->
    1    Year   1987-2008
    2    Month  1-12
    3    DayofMonth 1-31
    4    DayOfWeek  1 (Monday) - 7 (Sunday)
    5    DepTime    actual departure time (local, hhmm)
    6    CRSDepTime scheduled departure time (local, hhmm)
    7    ArrTime    actual arrival time (local, hhmm)
    8    CRSArrTime scheduled arrival time (local, hhmm)
    9    UniqueCarrier  unique carrier code
    10   FlightNum  flight number
    11   TailNum    plane tail number
    12   ActualElapsedTime  in minutes
    13   CRSElapsedTime in minutes
    14   AirTime    in minutes
    15   ArrDelay   arrival delay, in minutes
    16   DepDelay   departure delay, in minutes
    17   Origin origin IATA airport code
    18   Dest   destination IATA airport code
    19   Distance   in miles
    20   TaxiIn taxi in time, in minutes
    21   TaxiOut    taxi out time in minutes
    22   Cancelled  was the flight cancelled?
    23   CancellationCode   reason for cancellation (A = carrier, B = weather, C = NAS, D = security)
    24   Diverted   1 = yes, 0 = no
    25   CarrierDelay   in minutes
    26   WeatherDelay   in minutes
    27   NASDelay   in minutes
    28   SecurityDelay  in minutes
    29   LateAircraftDelay  in minutes

-   Download and rename the following auxiliary files:
    -   `airlines.csv` from <http://www.transtats.bts.gov/Download_Lookup.asp?Lookup=L_UNIQUE_CARRIERS>
    -   `airports.csv` from <https://raw.githubusercontent.com/jpatokal/openflights/master/data/airports.dat>
    -   `airplanes.csv` from [here](./airplanes.csv)
-   **Caution**
    -   Each file has a header.
    -   Years near 1987 has many `NA`s.
    -   `CancelationCode` is provided from 2003.
    -   Between 2003 and 2006, there are planes who's ages are negative. Ignore these data when analyzing.

Part 1. Constructing database (150 pts)
---------------------------------------

In this part of the project we construct a relational database using the `RSQLite` package, an R interface to the SQLite database management system. Recall Chapter 13 of the textbook where we study relational data, in which multiple tables are connected via keys. A (relational) database management system is software that provide fast and efficient access to large sets of tables. This system use a special language called structured query language (SQL) to query and modify the tables. Because the data you analyze is big (there are nearly 120 million records in total, and takes up 1.6 gigabytes of space compressed and 12 gigabytes when uncompressed.), R cannot load the dataset directly into the memory. Using a database is a must. Fortunately, you do not need to learn SQL for this project, as `dplyr` supports a variety of databases as a backend and translated `dplyr` verbs to SQL queries. However, keep in mind that you need a large disk space (approximately 50 GB).

1.  (20 pts) As an exercise, we create a simple SQLite database and query. Directions are as follows:
    -   Install `DBI` and `RSQLite` packages. SQLite is contained in `RSQLite`, so you do not need to install it.
    -   Create an empty SQLite database:

    ``` r
    library("DBI")
    library("RSQLite")
    con <- dbConnect(RSQLite::SQLite(), "employee.sqlite")
    str(con)
    ```

    -   Create two data frames:

    ``` r
    library("tidyverse")
    employees <- tibble(name   = c("Alice","Bob","Carol","Dave","Eve","Frank"),
                     email  = c("alice@company.com", "bob@company.com",
                                "carol@company.com", "dave@company.com",
                                "eve@company.com",   "frank@comany.com"),
                     salary = c(52000, 40000, 30000, 33000, 44000, 37000),
                     dept   = c("Accounting", "Accounting","Sales",
                                "Accounting","Sales","Sales"))
    phone <- tibble(name  = c("Bob", "Carol", "Eve", "Frank"),
                 phone = c("010-555-1111", "010-555-2222", "010-555-3333", "010-555-4444"))
    ```

    -   Write the data frames to the database:

    ``` r
    dbWriteTable(con, "employees", employees, overwrite = TRUE)
    dbWriteTable(con, "phone", phone, overwrite = TRUE)
    ```

    -   You can list the database tables:

    ``` r
    dbListTables(con)
    ```

    -   After creating a database, you may disconnect the connection:

    ``` r
    dbDisconnect(con)
    ```

    -   Now use `dplyr` to query the database. Of course, you need to reconnect to the database:

    ``` r
    recon <- dbConnect(RSQLite::SQLite(), "employee.sqlite")
    emp <- dplyr::tbl(recon, "employees")
    str(emp)
    ph <- dplyr::tbl(recon, "phone")
    str(ph)
    ```

    -   You can treat `emp` and `ph` as if they are just `tibble`s:

    ``` r
    addr <- emp %>%
        select(name, salary) %>%
        arrange(salary)
    addr
    left_join(emp,ph)
    ```

    -   You can do the similar task by directly feeding [SQL](https://en.wikipedia.org/wiki/SQL), the language for database, through DBI:

    ``` r
    res <- dbSendQuery(recon, "SELECT * FROM employees")
    dbFetch(res)
    dbClearResult(res)
    ```

    -   Don't forget to disconnect after you finish your job:

    ``` r
    dbDisconnect(recon)
    ```

    Now the task: provide all the outputs from the above R scripts.

2.  (40 pts) We start with small tables. Download files `airports.csv`, `airlines.csv`, and `airplanes.csv` as described above. Then write an R code snippet that creates a SQLite database containing these three tables in the same names as the filenames (without the extension). You may need some editing to read the csv files into R.

3.  (40 pts) Download files `1987.csv.bz2` through `2008.csv.bz2` from the Data Expo website above. Do *NOT* uncompress the files. `readr::read_csv()` supports reading bz2-compressed csv files directly. Using a `for` loop, write an R code snippet that creates a database table `flights` that contain all the data from 1987 through 2008. (Hint: study `append` option in `dbWriteTable()`.)

4.  (40 pts) The created database is more than 10 GB, and accessing the data is this large database takes long time even with a database management system. To speed up access to the data, you need to add indices. In this [page](http://stat-computing.org/dataexpo/2009/sqlite.html) of the Data Expo website, read the section "Adding indices," and write an R code snippet that adds indices to the database you've just created. (Hint. most of the codes in the Data Expo webpage are SQL queries.)

**Caution**: Indexing takes a lot of hard disk space. If your disk space is limited, then you may skip this step. However, you should nevertheless write a working code and submit.

1.  (10 pts) Put all the R code snippets you wrote into `flightdata.R` file so that `source()`ing this file will create the database at once.

Part 2. Basic Questions (150 pts)
---------------------------------

### Q1. Monthly traffic in three airports (60 pts)

1.  We first start by exploring the `airports` table. Using `dplyr::filter()`, find out which airports the codes "SNA", "SJC", and "SMF" belong to.

2.  Aggregate the counts of flights *to* all three of these airports at the monthly level (in the `flights` table) into a new data frame `airportcounts`. You may find `dplyr` functions `group_by()`, `summarise()`, `collect()`, and the pipe operator `%>%` useful.

3.  Add a new column to `airportcounts` by constructing a `Date` variable from the variables `Year` and `Month` (using helper functions from the `lubridate` package). Sort the rows in ascending order of dates.

4.  From `airportcounts`, generate a time series plot that plots the number of flights per month in each of the three airports in chronological order.

5.  Find the top ten months (like 2001-09) with the largest number of flights for each of the three airports.

### Q2. Finding reliable airlines (60 pts)

Which airline was most reliable flying from Chicago O'Hare (ORD) to Minneapolis/St. Paul (MSP) in Year 2005?

1.  Create a data frame `delays` that contains the average *arrival* delay for each day in 2005 for four airlines: United (UA), Northwest (NW), American (AA), and American Eagle (MQ). Your data frame must contain only necessary variables, to save the memory space.

2.  Compare the average delay of the four airlines by generating density plots comparing them in a single panel. In doing this, use a join function to provide the full names of the airlines in the legend of the plot. Which airline is the most reliable? which is the least?

### Q3. All flights (30 pts)

1.  Plot the count of *all* flights on Mondays in the year 2001. Explain the pattern you find in the visualization.

Part 3. Advanced Questions (100 pts)
------------------------------------

Provide a graphical summary to answer the following questions. These are intentionally vague in order to allow you to focus on different aspects of the data. We only provide some suggestions.

### Q1. When is the best time of day/day of week/time of year to fly to minimise delays? (50 pts)

Suggestions: cancelled flights do not possess delays. You may want to ignore negative delays.

### Q2. Do older planes suffer more delays? (50 pts)

Suggestions: you may want to find a correlation between the age of the plane and the departure delay. If the data size is too big to compute the correlation, you may try to sample a fraction from the dataset.
