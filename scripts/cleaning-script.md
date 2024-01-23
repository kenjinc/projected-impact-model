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
read.csv("/Users/kenjinchang/github/projected-impact-model/parent-datasets/enrollment_projections.csv") %>%
  as_tibble() 
```

    ## # A tibble: 73 × 20
    ##    Table.318…¹ X     X.1   X.2   X.3   X.4   X.5   X.6   X.7   X.8   X.9   X.10 
    ##    <chr>       <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
    ##  1 Year        Asso… Asso… Asso… Asso… Bach… "Bac… Bach… "Bac… Bach… "Bac… Bach…
    ##  2 Year        Total Males Fema… Perc… Total "Tot… Males "Mal… Fema… "Fem… Perc…
    ##  3 1           2     3     4     5     6     "6"   7     "7"   8     "8"   9    
    ##  4 1869-70     ---   ---   ---   ---   9,371 "\\2… 7,993 "\\2… 1,378 "\\2… 14.7 
    ##  5 1879-80     ---   ---   ---   ---   12,8… "\\2… 10,4… "\\2… 2,485 "\\2… 19.3 
    ##  6 1889-90     ---   ---   ---   ---   15,5… "\\2… 12,8… "\\2… 2,682 "\\2… 17.3 
    ##  7 1899-1900   ---   ---   ---   ---   27,4… "\\2… 22,1… "\\2… 5,237 "\\2… 19.1 
    ##  8 1909-10     ---   ---   ---   ---   37,1… "\\2… 28,7… "\\2… 8,437 "\\2… 22.7 
    ##  9 1919-20     ---   ---   ---   ---   48,6… "\\2… 31,9… "\\2… 16,6… "\\2… 34.2 
    ## 10 1929-30     ---   ---   ---   ---   122,… "\\2… 73,6… "\\2… 48,8… "\\2… 39.9 
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
    and `0` corresponding to enrollent totals that do not

``` r
read.csv("/Users/kenjinchang/github/projected-impact-model/parent-datasets/enrollment_projections.csv",skip=1) %>%
  as_tibble() %>%
  filter(!row_number() %in% c(2)) %>%
  select(!c(Associate.s.degrees.3,Bachelor.s.degrees.6,Master.s.degrees.3,Doctor.s.degrees.1..3)) %>%
  rename(year=Year,assoc_deg_tot=Associate.s.degrees,assoc_deg_m=Associate.s.degrees.1,assoc_deg_f=Associate.s.degrees.2,bach_deg_tot=Bachelor.s.degrees,bach_deg_m=Bachelor.s.degrees.2,bach_deg_f=Bachelor.s.degrees.4,mast_deg_tot=Master.s.degrees,mast_deg_m=Master.s.degrees.1,mast_deg_f=Master.s.degrees.2,doct_deg_tot=Doctor.s.degrees.1.,doct_deg_m=Doctor.s.degrees.1..1,doct_deg_f=Doctor.s.degrees.1..2) %>%
  filter(!row_number() %in% c(1,66,67,68,69,70,71)) %>%
  rename(bach_deg_tot_assoc=Bachelor.s.degrees.1,bach_deg_m_assoc=Bachelor.s.degrees.3,bach_deg_f_assoc=Bachelor.s.degrees.5) %>%
  mutate(bach_deg_tot_assoc=replace(bach_deg_tot_assoc, bach_deg_tot_assoc != "\\2\\","0")) %>%
  mutate(bach_deg_tot_assoc=replace(bach_deg_tot_assoc, bach_deg_tot_assoc == "\\2\\","1")) %>%
  mutate(bach_deg_m_assoc=replace(bach_deg_m_assoc, bach_deg_m_assoc != "\\2\\","0")) %>%
  mutate(bach_deg_m_assoc=replace(bach_deg_m_assoc, bach_deg_m_assoc == "\\2\\","1")) %>%
  mutate(bach_deg_f_assoc=replace(bach_deg_f_assoc, bach_deg_f_assoc != "\\2\\","0")) %>%
  mutate(bach_deg_f_assoc=replace(bach_deg_f_assoc, bach_deg_f_assoc == "\\2\\","1")) 
```

    ## # A tibble: 64 × 16
    ##    year  assoc…¹ assoc…² assoc…³ bach_…⁴ bach_…⁵ bach_…⁶ bach_…⁷ bach_…⁸ bach_…⁹
    ##    <chr> <chr>   <chr>   <chr>   <chr>   <chr>   <chr>   <chr>   <chr>   <chr>  
    ##  1 1869… ---     ---     ---     9,371   1       7,993   1       1,378   1      
    ##  2 1879… ---     ---     ---     12,896  1       10,411  1       2,485   1      
    ##  3 1889… ---     ---     ---     15,539  1       12,857  1       2,682   1      
    ##  4 1899… ---     ---     ---     27,410  1       22,173  1       5,237   1      
    ##  5 1909… ---     ---     ---     37,199  1       28,762  1       8,437   1      
    ##  6 1919… ---     ---     ---     48,622  1       31,980  1       16,642  1      
    ##  7 1929… ---     ---     ---     122,484 1       73,615  1       48,869  1      
    ##  8 1939… ---     ---     ---     186,500 1       109,546 1       76,954  1      
    ##  9 1949… ---     ---     ---     432,058 1       328,841 1       103,217 1      
    ## 10 1959… ---     ---     ---     392,440 1       254,063 1       138,377 1      
    ## # … with 54 more rows, 6 more variables: mast_deg_tot <chr>, mast_deg_m <chr>,
    ## #   mast_deg_f <chr>, doct_deg_tot <chr>, doct_deg_m <chr>, doct_deg_f <chr>,
    ## #   and abbreviated variable names ¹​assoc_deg_tot, ²​assoc_deg_m, ³​assoc_deg_f,
    ## #   ⁴​bach_deg_tot, ⁵​bach_deg_tot_assoc, ⁶​bach_deg_m, ⁷​bach_deg_m_assoc,
    ## #   ⁸​bach_deg_f, ⁹​bach_deg_f_assoc
