cleaning-script
================

## package loading

loading in the relevant packages CRAN packages.

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.4.1      ✔ purrr   0.3.4 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.1      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.2      ✔ forcats 0.5.2 
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(ggpubr)
```

## data loading

loading in the US enrollment data from the `parent-dataset` folder

``` r
as_tibble(read.csv("/Users/kenjinchang/github/projected-impact-model/parent-datasets/enrollment_projections.csv"))
```

    ## # A tibble: 73 × 20
    ##    Table.318…¹ X     X.1   X.2   X.3   X.4   X.5   X.6   X.7   X.8   X.9   X.10 
    ##    <chr>       <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
    ##  1 Year        "Ass… "Ass… "Ass… "Ass… Bach… "Bac… Bach… "Bac… Bach… "Bac… Bach…
    ##  2 Year        "Tot… "Mal… "Fem… "Per… Total "Tot… Males "Mal… Fema… "Fem… Perc…
    ##  3 1           "2"   "3"   "4"   "5"   6     "6"   7     "7"   8     "8"   9    
    ##  4 1869-70     ""    ""    ""    ""    9371… "\\2… 7993… "\\2… 1378… "\\2… 14.70
    ##  5 1879-80     ""    ""    ""    ""    1289… "\\2… 1041… "\\2… 2485… "\\2… 19.30
    ##  6 1889-90     ""    ""    ""    ""    1553… "\\2… 1285… "\\2… 2682… "\\2… 17.30
    ##  7 1899-1900   ""    ""    ""    ""    2741… "\\2… 2217… "\\2… 5237… "\\2… 19.10
    ##  8 1909-10     ""    ""    ""    ""    3719… "\\2… 2876… "\\2… 8437… "\\2… 22.70
    ##  9 1919-20     ""    ""    ""    ""    4862… "\\2… 3198… "\\2… 1664… "\\2… 34.20
    ## 10 1929-30     ""    ""    ""    ""    1224… "\\2… 7361… "\\2… 4886… "\\2… 39.90
    ## # … with 63 more rows, 8 more variables: X.11 <chr>, X.12 <chr>, X.13 <chr>,
    ## #   X.14 <chr>, X.15 <chr>, X.16 <chr>, X.17 <chr>, X.18 <chr>, and abbreviated
    ## #   variable name
    ## #   ¹​Table.318.10..Degrees.conferred.by.postsecondary.institutions..by.level.of.degree.and.sex.of.student..Selected.academic.years..1869.70.through.2031.32

performing the necessary cleaning procedures to make the parent dataset
suitable for our analyses

in order, we:

1.  omit the initial two rows that correspond to (1) the title of the
    table and and (2) the numeric column identifiers

2.  omit the degree-level percent-female variables

3.  rename the variables for brevity

4.  omit the row specifying the variable ID

5.  omit the source information and table notes from the tibble

6.  renamed the columns dichotomously indicating whether the
    corresponding bachelor-level enrollment value include degrees
    classified as master’s or doctor’s degree in later years

7.  changed column values within the the `bach_deg_tot_assoc`,
    `bach_deg_m_assoc`, and `bach_deg_f_assoc` from `\\2\\` to `1` and
    `0`, with `1` corresponding to enrollment totals that do incoude
    degrees classified as master’s or doctor’s degrees in later years
    and `0` corresponding to enrollment totals that do not

8.  changed columns from character strings to numeric strings

9.  replaced `NA` values with `0` values

10. generated new columns aggregating the number of degrees earned, both
    as a total and across sexes, between bachelor’s, master’s, and
    doctoral programs

11. generated new columns calculating the proportion of university
    degrees contributed by bachelor’s, master’s, and doctoral programs

``` r
enrollment_data <- as_tibble(read.csv("/Users/kenjinchang/github/projected-impact-model/parent-datasets/enrollment_projections.csv",skip=1)) %>%
  filter(!row_number() %in% c(2)) %>%
  select(!c(Associate.s.degrees.3,Bachelor.s.degrees.6,Master.s.degrees.3,Doctor.s.degrees.1..3)) %>%
  rename(acad.year=Year,assoc_deg_tot=Associate.s.degrees,assoc_deg_m=Associate.s.degrees.1,assoc_deg_f=Associate.s.degrees.2,bach_deg_tot=Bachelor.s.degrees,bach_deg_m=Bachelor.s.degrees.2,bach_deg_f=Bachelor.s.degrees.4,mast_deg_tot=Master.s.degrees,mast_deg_m=Master.s.degrees.1,mast_deg_f=Master.s.degrees.2,doct_deg_tot=Doctor.s.degrees.1.,doct_deg_m=Doctor.s.degrees.1..1,doct_deg_f=Doctor.s.degrees.1..2) %>%
  filter(!row_number() %in% c(1,66,67,68,69,70,71)) %>%
  rename(bach_deg_tot_assoc=Bachelor.s.degrees.1,bach_deg_m_assoc=Bachelor.s.degrees.3,bach_deg_f_assoc=Bachelor.s.degrees.5) %>%
  mutate(bach_deg_tot_assoc=replace(bach_deg_tot_assoc, bach_deg_tot_assoc != "\\2\\","0")) %>%
  mutate(bach_deg_tot_assoc=replace(bach_deg_tot_assoc, bach_deg_tot_assoc == "\\2\\","1")) %>%
  mutate(bach_deg_m_assoc=replace(bach_deg_m_assoc, bach_deg_m_assoc != "\\2\\","0")) %>%
  mutate(bach_deg_m_assoc=replace(bach_deg_m_assoc, bach_deg_m_assoc == "\\2\\","1")) %>%
  mutate(bach_deg_f_assoc=replace(bach_deg_f_assoc, bach_deg_f_assoc != "\\2\\","0")) %>%
  mutate(bach_deg_f_assoc=replace(bach_deg_f_assoc, bach_deg_f_assoc == "\\2\\","1")) %>%
  mutate(assoc_deg_tot=as.double(assoc_deg_tot)) %>%
  mutate(assoc_deg_m=as.double(assoc_deg_m)) %>%
  mutate(assoc_deg_f=as.double(assoc_deg_f)) %>%
  mutate(bach_deg_tot=as.double(bach_deg_tot)) %>%
  mutate(bach_deg_tot_assoc=as.double(bach_deg_tot_assoc)) %>%
  mutate(bach_deg_m=as.double(bach_deg_m)) %>%
  mutate(bach_deg_m_assoc=as.double(bach_deg_m_assoc)) %>%
  mutate(bach_deg_f=as.double(bach_deg_f)) %>%
  mutate(bach_deg_f_assoc=as.double(bach_deg_f_assoc)) %>%
  mutate(mast_deg_tot=as.double(mast_deg_tot)) %>%
  mutate(mast_deg_m=as.double(mast_deg_m)) %>%
  mutate(mast_deg_f=as.double(mast_deg_f)) %>%
  mutate(doct_deg_tot=as.double(doct_deg_tot)) %>%
  mutate(doct_deg_m=as.double(doct_deg_m)) %>%
  mutate(doct_deg_f=as.double(doct_deg_f)) %>%
  replace(is.na(.), 0) %>%
  mutate(univ_deg_tot=bach_deg_tot+mast_deg_tot+doct_deg_tot) %>%
  mutate(univ_deg_m=bach_deg_m+mast_deg_m+doct_deg_m) %>%
  mutate(univ_deg_f=bach_deg_f+mast_deg_f+doct_deg_f) %>%
  mutate(perc_bach_tot=bach_deg_tot/univ_deg_tot) %>%
  mutate(perc_mast_tot=mast_deg_tot/univ_deg_tot) %>%
  mutate(perc_doct_tot=doct_deg_tot/univ_deg_tot)
enrollment_data
```

    ## # A tibble: 64 × 22
    ##    acad.year assoc_deg…¹ assoc…² assoc…³ bach_…⁴ bach_…⁵ bach_…⁶ bach_…⁷ bach_…⁸
    ##    <chr>           <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
    ##  1 1869-70             0       0       0    9371       1    7993       1    1378
    ##  2 1879-80             0       0       0   12896       1   10411       1    2485
    ##  3 1889-90             0       0       0   15539       1   12857       1    2682
    ##  4 1899-1900           0       0       0   27410       1   22173       1    5237
    ##  5 1909-10             0       0       0   37199       1   28762       1    8437
    ##  6 1919-20             0       0       0   48622       1   31980       1   16642
    ##  7 1929-30             0       0       0  122484       1   73615       1   48869
    ##  8 1939-40             0       0       0  186500       1  109546       1   76954
    ##  9 1949-50             0       0       0  432058       1  328841       1  103217
    ## 10 1959-60             0       0       0  392440       1  254063       1  138377
    ## # … with 54 more rows, 13 more variables: bach_deg_f_assoc <dbl>,
    ## #   mast_deg_tot <dbl>, mast_deg_m <dbl>, mast_deg_f <dbl>, doct_deg_tot <dbl>,
    ## #   doct_deg_m <dbl>, doct_deg_f <dbl>, univ_deg_tot <dbl>, univ_deg_m <dbl>,
    ## #   univ_deg_f <dbl>, perc_bach_tot <dbl>, perc_mast_tot <dbl>,
    ## #   perc_doct_tot <dbl>, and abbreviated variable names ¹​assoc_deg_tot,
    ## #   ²​assoc_deg_m, ³​assoc_deg_f, ⁴​bach_deg_tot, ⁵​bach_deg_tot_assoc,
    ## #   ⁶​bach_deg_m, ⁷​bach_deg_m_assoc, ⁸​bach_deg_f

## preliminary data exploration

looking at the number of people enrolled in university over time

``` r
ggplot(enrollment_data,aes(x=acad.year,y=univ_deg_tot)) +
  geom_col() 
```

![](cleaning-script_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

may be useful to also include variable looking at proportion of national
population, to emphasize why these inerventions are partoculalry
important
